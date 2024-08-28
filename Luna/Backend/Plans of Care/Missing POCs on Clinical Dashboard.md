# Missing POCs on Clinical Dashboard Situations

## Missing because of Fax Setting

> Drs. are logging in to the dashborad to sign POCs. Tasks are missing from the dashboard. Dr. Fuancho as >60 outstanding POCs
> 
> I believe this is being triggered by the Luxe property "Signature Strategy Rule" when set to "fax_override".

Let's explore the code involved.

In Clinical Dashboard, pending plans of care come from Luxe. These are fetched using this endpoint:
```
api/v3/physician_dashboard/physicians/ID/unsigned_plans_of_care
```

In Edge, this endpoint is at:
```ruby
# app/controllers/api/v3/physician_dashboard/physicians/unsigned_plans_of_care_controller.rb

Api::V3::PhysicianDashboard::Physicians::UnsignedPlansOfCareController
```

In this controller, the query to grab POCs looks like this:
```ruby
pocs = PlanOfCare.includes(poc_includes)
	    .where(physician_id: params[:physician_id])
	    .waiting_for_response
```

The `waiting_for_response` scope:
```ruby
scope :waiting_for_response, lambda {
    where(needs_response: true, response_received: false)
      .where.not(
        PlanOfCareAction.where("plan_of_care_actions.plan_of_care_id = plans_of_care.id").arel.exists
      )
}
```

There's nothing suggesting a Fax setting being used to omit them.

### Plans of Care schema

```ruby
create_table "plans_of_care", id: :uuid, default: -> { "uuid_generate_v4()" }, force: :cascade do |t|
    t.bigint "chart_id", null: false
    t.string "communique_type", default: "fax", null: false
    t.string "messenger", null: false
    t.boolean "needs_response", default: false, null: false
    t.boolean "response_received", default: false, null: false
    t.string "comment", default: "", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.string "phaxio_id"
    t.uuid "physician_id"
    t.jsonb "frozen_fields", default: {}, null: false
    t.uuid "document_id"
    t.integer "original_referral_type"
    t.uuid "episode_id", null: false
    t.integer "page_count"
    t.uuid "outbound_transmission_attempt_id"

    t.index ["chart_id"], name: "index_plans_of_care_on_chart_id"
    t.index ["document_id"], name: "index_plans_of_care_on_document_id"
    t.index ["episode_id"], name: "index_plans_of_care_on_episode_id"
    t.index ["outbound_transmission_attempt_id"], name: "index_plans_of_care_on_outbound_transmission_attempt_id"
    t.index ["phaxio_id"], name: "index_plans_of_care_on_phaxio_id"
    t.index ["physician_id"], name: "index_plans_of_care_on_physician_id"
end
```

### Next Steps or Conclusion

Need to query some data in alpha to understand this plans_of_care table a bit more.