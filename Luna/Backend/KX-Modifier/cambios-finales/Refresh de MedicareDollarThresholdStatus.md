# Versión antigua: `PatientMedicareDollarThresholdStatus`

```ruby
module KxModifier
  # Create missing MedicareDollarThresholdStatus or delete unused ones.
  #
  # Unused ones happen when a Care Plan insurance changes to non-medicare or
  # care plan's appointments are transferred.
  class PatientMedicareDollarThresholdStatus
    def initialize(patient)
      @patient = patient
    end

    def refresh
      medicare_care_plans = patient.episodes.straight_medicare
      non_medicare_care_plans = patient.episodes.where.not(id: medicare_care_plans.ids)

      if medicare_care_plans.any?
        appts = medicare_care_plans.map { |mcp| mcp.appointments.active }.flatten
        appt_years = appts.map { |appt| appt.scheduled_date.year }.to_set

        mdts_statuses = find_medicare_statuses_in(appt_years)
        mdts_years = mdts_statuses.map { |status| status.effective_from.year }.to_set

        missing_years = appt_years - mdts_years

        if missing_years.length.positive?
          missing_years.each do |year|
            beginning_of_year = DateTime.new(year, 1, 1).beginning_of_year
            end_of_year = DateTime.new(year, 12, 31).end_of_year

            MedicareDollarThresholdStatus.create(
              patient: patient,
              effective_from: beginning_of_year,
              effective_until: end_of_year
            )
          end
        end
      else
        patient.medicare_dollar_threshold_statuses.destroy_all
      end

      return unless non_medicare_care_plans.any?

      non_medicare_appts = non_medicare_care_plans.map { |nmcp| nmcp.appointments.active }.flatten
      non_medicare_appt_years = non_medicare_appts.map { |appt| appt.scheduled_date.year }.to_set

      return unless appt_years.present?
      return unless non_medicare_appt_years.present?

      non_medicare_appt_years -= appt_years

      return if non_medicare_appt_years.empty?

      non_medicare_mdts_statuses = find_medicare_statuses_in(non_medicare_appt_years)

      return if appt_years == non_medicare_appt_years

      non_medicare_mdts_statuses.destroy_all
    end

    private

    attr_reader :patient

    def find_medicare_statuses_in(years)
      conditions = years.map do |year|
        "(effective_from = :start_#{year} AND effective_until = :end_#{year})"
      end.join(" OR ")

      params =
        years.index_with do |year|
          beginning_of_year = DateTime.new(year, 1, 1).beginning_of_year
          end_of_year = DateTime.new(year, 12, 31).end_of_year

          { "start_#{year}": beginning_of_year, "end_#{year}": end_of_year }
        end
        .values
        .reduce(&:merge)

      MedicareDollarThresholdStatus
        .where(patient: patient)
        .where(conditions, params)
    end
  end
end
```

# Versión refactor: `MedicareDollarThresholdStatusRefresherService`

```ruby
module KXModifier
  # Create missing MedicareDollarThresholdStatus or delete unused ones.
  #
  # Unused ones happen when a Care Plan insurance changes to non-medicare or
  # care plan's appointments are transferred.
  class MedicareDollarThresholdStatusRefresherService
    def initialize(patient)
      @patient = patient
    end

    def refresh
      medicare_care_plans, non_medicare_care_plans = patient.episodes.partition(&:straight_medicare?)

      medicare_care_plans.each do |care_plan|
        care_plan.appointments.active.each do |appointment|
          appointment.medicare_dollar_threshold_status = MedicareDollarThresholdStatus.find_or_create_by!(
            patient: patient,
            effective_from: appointment.scheduled_date.beginning_of_year,
            effective_until: appointment.scheduled_date.end_of_year
          )
          appointment.save! if appointment.medicare_dollar_threshold_status_id_changed?
        end
      end

      non_medicare_care_plans.each do |care_plan|
        # don't use destroy_all because it will attempt to destroy the appointments on the has_many through association
        care_plan.medicare_dollar_threshold_statuses.each(&:destroy)
      end

      medicare_appointments = medicare_care_plans.flat_map(&:appointments)
      active_medicare_appointment, inactive_medicare_appointment = medicare_appointments.partition(&:active?)

      active_medicare_appointment_years =
        active_medicare_appointment.map { |appt| appt.scheduled_date.year }.to_set
      inactive_medicare_appointment_years =
        inactive_medicare_appointment.map { |appt| appt.scheduled_date.year }.to_set

      unused_medicare_appointment_years = inactive_medicare_appointment_years - active_medicare_appointment_years

      unused_medicare_dollar_threshold_statuses =
        MedicareDollarThresholdStatus
        .find_in_years(unused_medicare_appointment_years)
        .where(patient: patient)

      unused_medicare_dollar_threshold_statuses.destroy_all
    end

    private

    attr_accessor :patient
  end
end
```