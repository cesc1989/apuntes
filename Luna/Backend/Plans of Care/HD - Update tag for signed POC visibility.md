# Help Desk - Update tag for signed POC visibility in Luxe

## Context

Request: Update tag in manage documents to ==make signed POCs visible to PT in the app==.

Detail: We would like ==signed POCs to automatically be visible to the PT in the app via documents==. They are currently tagged as “other” so they do not load into the app. **If the tag is changed to “patient documentation”, the PT will have visibility.**

POC == Plans of Care

## Exploration

All the things I've found while looking for the "patient documentation" tag.

- Found the `db/seeds/document_tags/patient_docuemtation.yml` file
- This file is used in `db/seeds.rb`
- It's needed for the `DocumentTagSeeder` class
- This is a seeder class for the `DocumentTag` model
- `PlanOfCare` is the model representation of `plans_of_care` table (POC)

> act-as-taggable-on -> https://github.com/mbleigh/acts-as-taggable-on

- Full list of Plans of Care in Clinical menu -> Plans of Care submenu - [Visit](http://localhost:3000/admin/plans_of_care)
- There's a "status" column
	- In `admin/clinical/plans_of_care.rb` you can see it uses `poc.status`. `poc.status` returns `[:approved, :modified, :rejected]`
- In the Plan of Care detail page you can access the attached `Document`
- In `PlanOfCareActionsController` the POC signature from Clinical Dashboard is handled
	- There's a worker: `PocDigitalSignatureWorker`. I suspect is this worker where the Tag should be added. However, the report says "currently tagged as _other_". I don't see that happening in this worker.
	- In `PatientFormsSignedTermsService` I did something similar. Added the tag like this: `document.tag_list.add("other")`


## Questions

How do I link an Episode (Care Plan) with a Plan of Care (POC)?

How do I find a Episode (Care Plan) with Plan of Care?


## Models

`PlanOfCare` has these associations:
```ruby
belongs_to :episode
belongs_to :document, optional: true
has_one :plan_of_care_action, required: false
```

`Episode` (Care Plan) has these associations:
```ruby
has_many :appointments, dependent: :destroy

has_many :charts, through: :appointments
has_many :plans_of_care, through: :charts
```

`Chart` model associations:
```ruby
has_many :plans_of_care, dependent: :destroy
has_many :plan_of_care_faxes, through: :plans_of_care

has_one :therapist, through: :appointment
has_one :episode, through: :appointment
```

The `Document` model is the one that handles tagging:
```ruby
class Document < ApplicationRecord
  acts_as_taggable_on :tags
end
```


## Schemas

`document_tags` schema:
```ruby
create_table "document_tags", id: :uuid, default: -> { "uuid_generate_v4()" }, force: :cascade do |t|
	t.string "name", null: false
	t.datetime "created_at", null: false
	t.datetime "updated_at", null: false
	t.string "auto_share_recipients", default: [], null: false, array: true
	t.integer "user_scope", default: 0, null: false
	t.string "label", null: false
	t.integer "position", default: 0, null: false
	t.index ["name"], name: "index_document_tags_on_name", unique: true
end
```

`plans_of_care` schema:
```ruby
create_table "plans_of_care", id: :uuid, default: -> { "uuid_generate_v4()" }, force: :cascade do |t|
	t.bigint "chart_id", null: false
	t.string "communique_type", default: "fax", null: false
	t.string "messenger", null: false
	t.boolean "needs_response", default: false, null: false
	t.boolean "response_received", default: false, null: false
	t.string "comment", default: "", null: false
	t.datetime "created_at", precision: nil, null: false
	t.datetime "updated_at", precision: nil, null: false
	t.string "phaxio_id"
	t.uuid "physician_id"
	t.jsonb "frozen_fields", default: {}, null: false
	t.uuid "document_id"
	t.integer "original_referral_type"
	t.uuid "episode_id", null: false
	t.integer "page_count"
	t.uuid "outbound_transmission_attempt_id"
	t.index ["chart_id"], name: "index_plans_of_care_on_chart_id"
	t.index ["created_at"], name: "index_plans_of_care_on_created_at"
	t.index ["document_id"], name: "index_plans_of_care_on_document_id"
	t.index ["episode_id"], name: "index_plans_of_care_on_episode_id"
	t.index ["outbound_transmission_attempt_id"], name: "index_plans_of_care_on_outbound_transmission_attempt_id"
	t.index ["phaxio_id"], name: "index_plans_of_care_on_phaxio_id"
	t.index ["physician_id"], name: "index_plans_of_care_on_physician_id"
end
```

`plan_of_care_actions` schema:
```ruby
create_table "plan_of_care_actions", id: :uuid, default: -> { "uuid_generate_v4()" }, force: :cascade do |t|
	t.uuid "plan_of_care_id", null: false
	t.integer "kind", null: false
	t.text "notes"
	t.jsonb "metadata", default: {}, null: false
	t.datetime "created_at", null: false
	t.datetime "updated_at", null: false
	t.index ["plan_of_care_id"], name: "index_plan_of_care_actions_on_plan_of_care_id", unique: true
end
```