# Browsing ActionView for missing URL in form builder

In rails 6.1:
```ruby
<%= f.options[:url].inspect %>

"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9"
```

In rails 7.0:
```ruby
<%= f.options[:url].inspect %>

nil
```

Why? Let's try to find the reason.

# Navigating Form Builder in Rails 6.1

In FormHelper:
```ruby
[450, 459] in /Users/francisco/.gem/ruby/3.1.0/gems/actionview-6.1.7.8/lib/action_view/helpers/form_helper.rb
   450:         output  = capture(builder, &block)
   451:         html_options[:multipart] ||= builder.multipart?
   452:
   453:         html_options = html_options_for_form(options[:url] || {}, html_options)
   454:         debugger
=> 455:         form_tag_with_body(html_options, output)
   456:       end
   457:
   458:       def apply_form_for_options!(record, object, options) #:nodoc:
   459:         object = convert_to_model(object)
(byebug) options[:url]
"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9"
(byebug) html_options
{"class"=>"formtastic care_plan_form", "id"=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", "method"=>:patch, "novalidate"=>false, "authenticity_token"=>nil, "action"=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", "accept-charset"=>"UTF-8"}
(byebug) html_options
{"class"=>"formtastic care_plan_form", "id"=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", "method"=>:patch, "novalidate"=>false, "authenticity_token"=>nil, "action"=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", "accept-charset"=>"UTF-8"}
(byebug) html_options_for_form(options[:url] || {}, html_options)
{"class"=>"formtastic care_plan_form", "id"=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", "method"=>:patch, "novalidate"=>false, "authenticity_token"=>nil, "action"=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", "accept-charset"=>"UTF-8"}
```

In FormTagHelper:
```ruby
[885, 894] in /Users/francisco/.gem/ruby/3.1.0/gems/actionview-6.1.7.8/lib/action_view/helpers/form_tag_helper.rb
   885:           tag(:form, html_options, true) + extra_tags
   886:         end
   887:
   888:         def form_tag_with_body(html_options, content)
   889:           debugger
=> 890:           output = form_tag_html(html_options)
   891:           output << content
   892:           output.safe_concat("</form>")
   893:         end
   894:
(byebug) html_options
{"class"=>"formtastic care_plan_form", "id"=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", "method"=>:patch, "novalidate"=>false, "authenticity_token"=>nil, "action"=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", "accept-charset"=>"UTF-8"}
(byebug) form_tag_html(html_options)
"<form class=\"formtastic care_plan_form\" id=\"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9\" action=\"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9\" accept-charset=\"UTF-8\" method=\"post\"><input type=\"hidden\" name=\"authenticity_token\" value=\"SlwFmS3ZovbbvC0UCqLMR+y+i1FJ9/3yvulPDZwIfih0cgIkr32jVc8JCdMq2ZwLMIGsOB3bx7JP3UsDhQK27Q==\" autocomplete=\"off\" />"
```

In FormTagHelper, there's the private method `html_options_for_form(url_for_options, options)`. It defines:
```ruby
# The following URL is unescaped, this is just a hash of options, and it is the
# responsibility of the caller to escape all the values.
html_options["action"]  = url_for(url_for_options)
html_options["accept-charset"] = "UTF-8"
```

When inspecting closely:
```ruby
[835, 844] in /Users/francisco/.gem/ruby/3.1.0/gems/actionview-6.1.7.8/lib/action_view/helpers/form_tag_helper.rb
   835:             html_options["action"]  = url_for(url_for_options)
   836:             html_options["accept-charset"] = "UTF-8"
   837:
   838:             html_options["data-remote"] = true if html_options.delete("remote")
   839: debugger
=> 840:             if html_options["data-remote"] &&
   841:                !embed_authenticity_token_in_remote_forms &&
   842:                html_options["authenticity_token"].blank?
   843:               # The authenticity token is taken from the meta tag in this case
   844:               html_options["authenticity_token"] = false
(byebug) url_for_options
"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9"
(byebug) options
{:class=>"formtastic care_plan_form", :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", :method=>:patch, :novalidate=>false, :authenticity_token=>nil, :multipart=>nil}
(byebug) html_options
{"class"=>"formtastic care_plan_form", "id"=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", "method"=>:patch, "novalidate"=>false, "authenticity_token"=>nil, "action"=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", "accept-charset"=>"UTF-8"}
```

# Formtastic up to ActionView in Rails 6.1

## In Formtastic

Method `initialize(object_name, object, template, options)`
```ruby
[95, 104] in /Users/francisco/.gem/ruby/3.1.0/gems/formtastic-4.0.0/lib/formtastic/form_builder.rb
    95:     end
    96:
    97:     def initialize(object_name, object, template, options)
    98:       super
    99: debugger
=> 100:       if respond_to?('multipart=') && options.is_a?(Hash) && options[:html]
   101:         self.multipart = options[:html][:multipart]
   102:       end
   103:     end
   104:
(byebug) options
{:url=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", :builder=>Formtastic::FormBuilder, :html=>{:class=>"formtastic care_plan_form", :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", :method=>:patch, :novalidate=>false, :authenticity_token=>nil}, :custom_namespace=>nil}
```

# Navigating Form Builder in Rails 7.0

## In FormHelper

Method `form_for(record, options = {}, &block)`:
```ruby
[455, 464] in /Users/francisco/.gem/ruby/3.1.0/gems/actionview-7.0.7/lib/action_view/helpers/form_helper.rb
   455:         options[:scope]                               = object_name
   456:         options[:local]                               = !remote
   457:         options[:skip_default_ids]                    = false
   458:         options[:allow_method_names_outside_object]   = options.fetch(:allow_method_names_outside_object, false)
   459: debugger
=> 460:         form_with(**options, &block)
   461:       end
   462:
   463:       def apply_form_for_options!(object, options) # :nodoc:
   464:         object = convert_to_model(object)
(byebug) options
{:url=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", :builder=>Formtastic::FormBuilder, :html=>{:class=>"formtastic care_plan_form", :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", :novalidate=>false}, :custom_namespace=>nil, :model=>[:admin, #<CarePlanForm], :scope=>"care_plan_form", :local=>true, :skip_default_ids=>false, :allow_method_names_outside_object=>false}
(byebug) apply_form_for_options!(object, options)
{:class=>"formtastic care_plan_form", :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", :novalidate=>false}
```

Method `form_with(model: nil, scope: nil, url: nil, format: nil, **options, &block)`:
```ruby
[768, 777] in /Users/francisco/.gem/ruby/3.1.0/gems/actionview-7.0.7/lib/action_view/helpers/form_helper.rb
   768:           output  = capture(builder, &block)
   769:           options[:multipart] ||= builder.multipart?
   770:
   771:           html_options = html_options_for_form_with(url, model, **options)
   772:           debugger
=> 773:           form_tag_with_body(html_options, output)
   774:         else
   775:           html_options = html_options_for_form_with(url, model, **options)
   776:           form_tag_html(html_options)
   777:         end
(byebug) options
{:allow_method_names_outside_object=>false, :skip_default_ids=>false, :builder=>Formtastic::FormBuilder, :html=>{:class=>"formtastic care_plan_form", :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", :novalidate=>false}, :custom_namespace=>nil, :local=>true, :multipart=>nil}
(byebug) url
"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9"
(byebug) html_options
{"class"=>"formtastic care_plan_form", "id"=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", "novalidate"=>false, "method"=>:patch, "action"=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", "accept-charset"=>"UTF-8"}
```

## In FormTagHelper

Method `form_tag_with_body`:
```ruby
[982, 991] in /Users/francisco/.gem/ruby/3.1.0/gems/actionview-7.0.7/lib/action_view/helpers/form_tag_helper.rb
   982:           tag(:form, html_options, true) + extra_tags
   983:         end
   984:
   985:         def form_tag_with_body(html_options, content)
   986:           debugger
=> 987:           output = form_tag_html(html_options)
   988:           output << content.to_s if content
   989:           output.safe_concat("</form>")
   990:         end
   991:
(byebug) html_options
{"class"=>"formtastic care_plan_form", "id"=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", "novalidate"=>false, "method"=>:patch, "action"=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", "accept-charset"=>"UTF-8"}
```

# Formtastic up to ActionView in Rails 7.0

## In Formtastic

Method `initialize(object_name, object, template, options)`
```ruby
[95, 104] in /Users/francisco/.gem/ruby/3.1.0/gems/formtastic-4.0.0/lib/formtastic/form_builder.rb
    95:     end
    96:
    97:     def initialize(object_name, object, template, options)
    98:       super
    99: debugger
=> 100:       if respond_to?('multipart=') && options.is_a?(Hash) && options[:html]
   101:         self.multipart = options[:html][:multipart]
   102:       end
   103:     end
   104:
(byebug) options
{:allow_method_names_outside_object=>false, :skip_default_ids=>false, :builder=>Formtastic::FormBuilder, :html=>{:class=>"formtastic care_plan_form", :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", :novalidate=>false}, :custom_namespace=>nil, :local=>true}
```

## In ActionView

Method `instantiate_builder(record_name, record_object, options)`
```ruby
[1599, 1608] in /Users/francisco/.gem/ruby/3.1.0/gems/actionview-7.0.7/lib/action_view/helpers/form_helper.rb
   1599:             object = record_name
   1600:             object_name = model_name_from_record_or_class(object).param_key if object
   1601:           end
   1602:
   1603:           builder = options[:builder] || default_form_builder_class
=> 1604:           builder.new(object_name, object, self, options)
   1605:         end
   1606:
   1607:         def default_form_builder_class
   1608:           builder = default_form_builder || ActionView::Base.default_form_builder
(byebug) options
{:allow_method_names_outside_object=>false, :skip_default_ids=>false, :builder=>Formtastic::FormBuilder, :html=>{:class=>"formtastic care_plan_form", :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", :novalidate=>false}, :custom_namespace=>nil, :local=>true}
```

Method `form_with(model: nil, scope: nil, url: nil, format: nil, **options, &block)`
```ruby
[762, 771] in /Users/francisco/.gem/ruby/3.1.0/gems/actionview-7.0.7/lib/action_view/helpers/form_helper.rb
   762:           model   = _object_for_form_builder(model)
   763:           scope ||= model_name_from_record_or_class(model).param_key
   764:         end
   765:
   766:         if block_given?
=> 767:           builder = instantiate_builder(scope, model, options)
   768:           output  = capture(builder, &block)
   769:           options[:multipart] ||= builder.multipart?
   770:
   771:           html_options = html_options_for_form_with(url, model, **options)
(byebug) options
{:allow_method_names_outside_object=>false, :skip_default_ids=>false, :builder=>Formtastic::FormBuilder, :html=>{:class=>"formtastic care_plan_form", :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", :novalidate=>false}, :custom_namespace=>nil, :local=>true}
```

Method `form_for(record, options = {}, &block)`:
```ruby
[455, 464] in /Users/francisco/.gem/ruby/3.1.0/gems/actionview-7.0.7/lib/action_view/helpers/form_helper.rb
   455:         options[:scope]                               = object_name
   456:         options[:local]                               = !remote
   457:         options[:skip_default_ids]                    = false
   458:         options[:allow_method_names_outside_object]   = options.fetch(:allow_method_names_outside_object, false)
   459:
=> 460:         form_with(**options, &block)
   461:       end
   462:
   463:       def apply_form_for_options!(object, options) # :nodoc:
   464:         object = convert_to_model(object)
(byebug) options
{:url=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9", :builder=>Formtastic::FormBuilder, :html=>{:class=>"formtastic care_plan_form", :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9", :novalidate=>false}, :custom_namespace=>nil, :model=>[:admin, CarePlanForm], :scope=>"care_plan_form", :local=>true, :skip_default_ids=>false, :allow_method_names_outside_object=>false}
```