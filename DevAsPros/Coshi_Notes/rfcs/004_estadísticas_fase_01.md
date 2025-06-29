# Statistics Implementation Plan

Generado por Claude Code.

## Overview

This document outlines the implementation plan for two key statistics features:

1. **Most active themes by comment count**
2. **Most active channels by engagement**

## Current Application Context

- **Rails 7.1.4** with modern stack (Sidekiq, Devise, Active Storage)
- **Discussion platform** with Users → Channels → Themes → Comments hierarchy
- **No existing statistics system** - clean slate implementation
- **Admin infrastructure** partially in place (admin users, protected routes)
- **Solid testing framework** (RSpec, FactoryBot, Capybara)

## Implementation Strategy

### Phase 1: Core Statistics Foundation (1-2 days)

#### 1.1 Database Schema

Create statistics tables to store aggregated data:

```ruby
# db/migrate/xxx_create_statistics_tables.rb
class CreateStatisticsTables < ActiveRecord::Migration[7.1]
  def change
    create_table :theme_statistics do |t|
      t.references :theme, null: false, foreign_key: true
      t.integer :comment_count, default: 0
      t.integer :unique_commenters, default: 0
      t.datetime :last_activity_at
      t.timestamps
    end

    create_table :channel_statistics do |t|
      t.references :channel, null: false, foreign_key: true
      t.integer :theme_count, default: 0
      t.integer :comment_count, default: 0
      t.integer :unique_participants, default: 0
      t.datetime :last_activity_at
      t.timestamps
    end

    add_index :theme_statistics, :comment_count
    add_index :channel_statistics, [:comment_count, :theme_count]
  end
end
```

#### 1.2 Statistics Models

```ruby
# app/models/theme_statistic.rb
class ThemeStatistic < ApplicationRecord
  belongs_to :theme
  
  scope :most_active, -> { order(comment_count: :desc) }
  scope :recent_activity, -> { where('last_activity_at > ?', 1.week.ago) }
end

# app/models/channel_statistic.rb
class ChannelStatistic < ApplicationRecord
  belongs_to :channel
  
  scope :most_engaging, -> { order(engagement_score: :desc) }
  
  def engagement_score
    # Weighted score: comments * 1 + themes * 3 + unique_participants * 2
    comment_count + (theme_count * 3) + (unique_participants * 2)
  end
end
```

#### 1.3 Statistics Service

```ruby
# app/services/statistics_service.rb
class StatisticsService
  def self.calculate_theme_statistics(theme_id = nil)
    themes = theme_id ? Theme.where(id: theme_id) : Theme.all
    
    themes.find_each do |theme|
      stats = ThemeStatistic.find_or_initialize_by(theme: theme)
      stats.update!(
        comment_count: theme.comments.count,
        unique_commenters: theme.comments.distinct.count(:user_id),
        last_activity_at: theme.comments.maximum(:created_at) || theme.created_at
      )
    end
  end
  
  def self.calculate_channel_statistics(channel_id = nil)
    channels = channel_id ? Channel.where(id: channel_id) : Channel.all
    
    channels.find_each do |channel|
      stats = ChannelStatistic.find_or_initialize_by(channel: channel)
      
      theme_count = channel.themes.count
      comment_count = Comment.joins(:theme).where(themes: { channel: channel }).count
      unique_participants = User.joins(:comments)
                               .joins("JOIN themes ON comments.theme_id = themes.id")
                               .where(themes: { channel: channel })
                               .distinct.count
      
      stats.update!(
        theme_count: theme_count,
        comment_count: comment_count,
        unique_participants: unique_participants,
        last_activity_at: [
          channel.themes.maximum(:created_at),
          Comment.joins(:theme).where(themes: { channel: channel }).maximum(:created_at)
        ].compact.max || channel.created_at
      )
    end
  end
end
```

### Phase 2: Real-time Updates (1 day)

#### 2.1 Model Callbacks

```ruby
# Update existing models to trigger statistics updates

# app/models/comment.rb
class Comment < ApplicationRecord
  after_create :update_statistics
  after_destroy :update_statistics
  
  private
  
  def update_statistics
    UpdateStatisticsJob.perform_later(theme_id: theme_id)
  end
end

# app/models/theme.rb  
class Theme < ApplicationRecord
  after_create :update_channel_statistics
  after_update :update_channel_statistics, if: :saved_change_to_closed?
  
  private
  
  def update_channel_statistics
    UpdateChannelStatisticsJob.perform_later(channel_id: channel_id)
  end
end
```

#### 2.2 Background Jobs

```ruby
# app/jobs/update_statistics_job.rb
class UpdateStatisticsJob < ApplicationJob
  queue_as :default
  
  def perform(theme_id:)
    StatisticsService.calculate_theme_statistics(theme_id)
    
    # Also update channel statistics for the theme's channel
    theme = Theme.find(theme_id)
    StatisticsService.calculate_channel_statistics(theme.channel_id)
  end
end

# app/jobs/update_channel_statistics_job.rb
class UpdateChannelStatisticsJob < ApplicationJob
  queue_as :default
  
  def perform(channel_id:)
    StatisticsService.calculate_channel_statistics(channel_id)
  end
end
```

### Phase 3: Admin Dashboard (2-3 days)

#### 3.1 Routes

```ruby
# config/routes.rb
namespace :admin do
  resources :statistics, only: [:index] do
    collection do
      get :themes
      get :channels
      get :export
    end
  end
end
```

#### 3.2 Controller

```ruby
# app/controllers/admin/statistics_controller.rb
class Admin::StatisticsController < ApplicationController
  before_action :authenticate_user!
  before_action :ensure_admin!
  
  def index
    @theme_stats = most_active_themes
    @channel_stats = most_active_channels
  end
  
  def themes
    @themes = most_active_themes(params[:limit]&.to_i || 50)
    render json: @themes.map { |ts| theme_stats_json(ts) }
  end
  
  def channels
    @channels = most_active_channels(params[:limit]&.to_i || 50)
    render json: @channels.map { |cs| channel_stats_json(cs) }
  end
  
  private
  
  def most_active_themes(limit = 10)
    ThemeStatistic.joins(:theme)
                  .includes(theme: [:channel])
                  .most_active
                  .limit(limit)
  end
  
  def most_active_channels(limit = 10)
    ChannelStatistic.joins(:channel)
                    .includes(:channel)
                    .most_engaging
                    .limit(limit)
  end
  
  def theme_stats_json(theme_stat)
    {
      id: theme_stat.theme.id,
      name: theme_stat.theme.name,
      channel_name: theme_stat.theme.channel.name,
      comment_count: theme_stat.comment_count,
      unique_commenters: theme_stat.unique_commenters,
      last_activity: theme_stat.last_activity_at
    }
  end
  
  def channel_stats_json(channel_stat)
    {
      id: channel_stat.channel.id,
      name: channel_stat.channel.name,
      theme_count: channel_stat.theme_count,
      comment_count: channel_stat.comment_count,
      unique_participants: channel_stat.unique_participants,
      engagement_score: channel_stat.engagement_score,
      last_activity: channel_stat.last_activity_at
    }
  end
  
  def ensure_admin!
    redirect_to root_path unless current_user.admin?
  end
end
```

#### 3.3 Views

```erb
<%# app/views/admin/statistics/index.html.erb %>

<div class="container-fluid">
  <div class="row">
    <div class="col-12">
      <h1>Statistics Dashboard</h1>
    </div>
  </div>
  
  <div class="row">
    <div class="col-md-6">
      <div class="card">
        <div class="card-header">
          <h3>Most Active Themes</h3>
        </div>
        <div class="card-body">
          <canvas id="themesChart" width="400" height="200"></canvas>
          <div class="table-responsive mt-3">
            <table class="table table-striped">
              <thead>
                <tr>
                  <th>Theme</th>
                  <th>Channel</th>
                  <th>Comments</th>
                  <th>Participants</th>
                </tr>
              </thead>
              <tbody>
                <% @theme_stats.each do |stat| %>
                  <tr>
                    <td><%= link_to stat.theme.name, channel_theme_path(stat.theme.channel, stat.theme) %></td>
                    <td><%= stat.theme.channel.name %></td>
                    <td><%= stat.comment_count %></td>
                    <td><%= stat.unique_commenters %></td>
                  </tr>
                <% end %>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
    
    <div class="col-md-6">
      <div class="card">
        <div class="card-header">
          <h3>Most Active Channels</h3>
        </div>
        <div class="card-body">
          <canvas id="channelsChart" width="400" height="200"></canvas>
          <div class="table-responsive mt-3">
            <table class="table table-striped">
              <thead>
                <tr>
                  <th>Channel</th>
                  <th>Themes</th>
                  <th>Comments</th>
                  <th>Engagement Score</th>
                </tr>
              </thead>
              <tbody>
                <% @channel_stats.each do |stat| %>
                  <tr>
                    <td><%= link_to stat.channel.name, channel_path(stat.channel) %></td>
                    <td><%= stat.theme_count %></td>
                    <td><%= stat.comment_count %></td>
                    <td><%= stat.engagement_score %></td>
                  </tr>
                <% end %>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', function() {
  // Fetch and render charts
  fetch('/admin/statistics/themes')
    .then(response => response.json())
    .then(data => renderThemesChart(data));
    
  fetch('/admin/statistics/channels')
    .then(response => response.json())
    .then(data => renderChannelsChart(data));
});

function renderThemesChart(data) {
  const ctx = document.getElementById('themesChart').getContext('2d');
  new Chart(ctx, {
    type: 'bar',
    data: {
      labels: data.slice(0, 10).map(t => t.name),
      datasets: [{
        label: 'Comments',
        data: data.slice(0, 10).map(t => t.comment_count),
        backgroundColor: 'rgba(54, 162, 235, 0.2)',
        borderColor: 'rgba(54, 162, 235, 1)',
        borderWidth: 1
      }]
    },
    options: {
      responsive: true,
      scales: {
        y: { beginAtZero: true }
      }
    }
  });
}

function renderChannelsChart(data) {
  const ctx = document.getElementById('channelsChart').getContext('2d');
  new Chart(ctx, {
    type: 'bar',
    data: {
      labels: data.slice(0, 10).map(c => c.name),
      datasets: [{
        label: 'Engagement Score',
        data: data.slice(0, 10).map(c => c.engagement_score),
        backgroundColor: 'rgba(255, 99, 132, 0.2)',
        borderColor: 'rgba(255, 99, 132, 1)',
        borderWidth: 1
      }]
    },
    options: {
      responsive: true,
      scales: {
        y: { beginAtZero: true }
      }
    }
  });
}
</script>
```

### Phase 4: Initial Data Population & Maintenance (1 day)

#### 4.1 Rake Tasks

```ruby
# lib/tasks/statistics.rake
namespace :statistics do
  desc "Calculate all statistics"
  task calculate_all: :environment do
    puts "Calculating theme statistics..."
    StatisticsService.calculate_theme_statistics
    
    puts "Calculating channel statistics..."
    StatisticsService.calculate_channel_statistics
    
    puts "Statistics calculation complete!"
  end
  
  desc "Recalculate statistics for a specific theme"
  task :recalculate_theme, [:theme_id] => :environment do |task, args|
    StatisticsService.calculate_theme_statistics(args[:theme_id])
    puts "Theme #{args[:theme_id]} statistics updated!"
  end
end
```

#### 4.2 Scheduled Jobs (Optional)

```ruby
# app/jobs/daily_statistics_job.rb
class DailyStatisticsJob < ApplicationJob
  queue_as :default
  
  def perform
    StatisticsService.calculate_theme_statistics
    StatisticsService.calculate_channel_statistics
  end
end

# Schedule with cron or sidekiq-cron
# 0 2 * * * (daily at 2 AM)
```

### Phase 5: Testing (1-2 days)

#### 5.1 Model Tests

```ruby
# spec/models/theme_statistic_spec.rb
# spec/models/channel_statistic_spec.rb
# spec/services/statistics_service_spec.rb
```

#### 5.2 Controller Tests

```ruby
# spec/controllers/admin/statistics_controller_spec.rb
```

#### 5.3 Integration Tests

```ruby
# spec/system/admin/statistics_spec.rb
```

## Implementation Timeline

| Phase | Duration | Key Deliverables |
|-------|----------|------------------|
| **Phase 1** | 1-2 days | Statistics models, service, database schema |
| **Phase 2** | 1 day | Real-time updates via callbacks and jobs |
| **Phase 3** | 2-3 days | Admin dashboard with charts and tables |
| **Phase 4** | 1 day | Data population and maintenance tasks |
| **Phase 5** | 1-2 days | Comprehensive testing |
| **Total** | **6-9 days** | Full statistics implementation |

## Dependencies & Requirements

### Gems to Add
```ruby
# Gemfile
gem 'chartkick'  # For easy chart integration
gem 'groupdate'  # For time-based grouping (if needed later)
```

### JavaScript Libraries

- Chart.js (already available via CDN or add to importmap)

### Database Considerations

- Add indexes for performance on large datasets
- Consider partitioning for very large comment tables
- Monitor query performance with statistics queries

## Success Metrics

1. **Performance**: Statistics queries execute in < 100ms
2. **Accuracy**: Real-time updates reflect changes within 5 seconds
3. **Usability**: Admin dashboard loads in < 2 seconds
4. **Reliability**: Background jobs process without errors
5. **Testing**: > 90% test coverage on statistics code

## Future Enhancements

1. **Time-based filtering** (last 7 days, month, year)
2. **Export functionality** (CSV, PDF reports)
3. **User engagement statistics**
4. **Trend analysis and forecasting**
5. **Real-time dashboard updates** (via ActionCable)
6. **API endpoints** for external integrations

## Risk Mitigation

1. **Performance**: Use database indexes and consider caching
2. **Data consistency**: Use database transactions for statistics updates
3. **Scalability**: Design for background processing from the start
4. **Maintenance**: Include rake tasks for data recalculation

This plan provides a solid foundation for implementing the requested statistics while maintaining the application's existing architecture and patterns.
