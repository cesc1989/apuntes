# ✅ Progress Forms Mobile API

## Sections

[ASES, LEFS, ODI, NDI, PSFS] Forms:

- Pain Scale → Section 1
- Aggravating Activities → Section 2
- Functional Limitations → Section 3
    - Amount of questions depends on form type.
    - ODI, NID: Ten questions.
    - ASES: Eleven questions.
    - LEFS: Twenty questions.
    - PSFS: Zero questions.

HOOS JR Forms:

- Pain Scale → Section 1
- Aggravating Activities → Section 2
- Functional Limitations
    - Pain → Section 3. *Two questions*.
    - Function → Section 4. *Four questions*.

KOOS JR Forms:

- Pain Scale → Section 1
- Aggravating Activities → Section 2
- Functional Limitations
    - Stiffness → Section 3. *One question*.
    - Pain → Section 4. *Four questions*.
    - Function → Section 5. *Two questions*.
# Endpoints
## Get Form Status Endpoint

Requests

- `GET /v2/patient/care-plans/<care-plan-id>/form-status`
- `GET /v2/therapist/care-plans/<care-plan-id>/form-status`


- `status` can be “complete” or “incomplete”
- `current_section` can be “pain”, “psfs”, or “functional”

Response object for **latest intake and progress form**

    {
      "form_status": {
        "intake": {
          "id": "5851c1a0-249e-4c2e-aac1-b9ac038307b2", 
          "number": 1,
          "status": "complete", 
          "created_at": "2020-04-13", 
          "completed_at": "2020-04-16",
          "web_url": "https://example.com/intake-form-responses", 
          "transition_url": null
        }, 
        "progress": {
          "id": "bc4e3c44-281f-4bde-b0a1-e682a71d4443", 
          "number": 2, 
          "status": "incomplete", 
          "created_at": "2020-05-13", 
          "completed_at": null, 
          "web_url": "https://example.com/progress-form-questions", 
          "transition_url": "https://example.com/v2/progress-form-drafts/12345?section=1"
        }
      }
    }


## Get Pain Scale Section Components

**Section 1**
Components:

- Header
- Numeric Radio Button Input

Request using `transition_url` from status endpoint.

- `GET "https://example.com/v2/progress-form-drafts/12345?section=1"`

Response object

    {
      "draft": {
        "section": 1, 
        "sections_total": 5, 
        "components": [
          {
            "id": "5a0653fe-8549-4722-9b8e-566755eefc57", 
            "type": "header", 
            "data": {
              "label": "Your Pain Scale", 
              "body": "How much pain did you have in the past *7 days*? Please select a number on the scale."
            }
          }, 
          {
            "id": "09764419-d210-46e5-af25-2af1b94b6d87",
            "type": "numeric_radio_button_input",
            "data": { 
              "lower_value": 0, 
              "upper_value": 10,
              "lower_label": "No pain", 
              "upper_label": "Unimaginable pain", 
              "optimal_value": 0, 
              "value": "null", 
              "previous_value": 5, 
              "previous_value_date": "2020-04-29", 
              "regression_warning_text": "Your pain score worsened (last submission: 5).", 
              "regression_warning_subtext": "Please double check that this is correct."
            }, 
            "required": true
          }
        ],
        "previous_url": null, 
        "next_url": "https://example.com/v2/progress-form-drafts/<draft-id>?section=2"
      }
    }


## Save Pain Scale Section Draft

Request endpoint is current URL.

- `PUT "https://example.com/v2/progress-form-drafts/12345?section=1"`

Request object sends the Component ID and the user-selected value.

    {
      "draft_values": [
        {
          "component_id": "09764419-d210-46e5-af25-2af1b94b6d87",
          "value": 4
        }
      ]
    }

Response should be enough with the OK or CREATED status code.


## Get Aggravating Activities Section Components

**Section 2**
Components:

- Header
- Numeric Slider List Input

Request using section 1  `next_url` value.

- `GET "https://example.com/v2/progress-form-drafts/<draft-id>?section=2`

Response object.


> Remember: aggravating activities are preloaded form the first(intake) Progress Form.


    {
      "draft": {
        "section": 2,
        "sections_total": 5,
        "components": [
          {
            "id": "48ca7080-ab31-497d-941d-7eb50baaca7b", 
            "type": "header", 
            "data": {
              "label": "Aggravating Activities", 
              "body": "List 3 activities that are difficult for you or you are unable to do as a result of your injury. Use the scale next to each activity to show your current ability level. Please note, 10 means you have recovered and are able to do the activity as before the injury."
            }
          }, 
          {
            "id": "d5b720f4-b7e8-406b-876a-73787c603622",
            "type": "numeric_slider_list_input", 
            "data": {
              "title": "Activity", 
              "lower_value": 0,
              "upper_value": 10,
              "lower_label": "Unable to Perform Activity",
              "upper_label": "Able to Perform Activity at the Same Level as Before Injury or Problem",
              "optimal_value": 10,
              "items": [
                {
                  "label": "Ascending Stairs", 
                  "value": null,
                  "previous_value": 2, 
                  "previous_value_date": "2020-04-29", 
                  "required": true,
                  "regression_warning_text": "Your pain score worsened (last submission: 2).",
                  "regression_warning_subtext": "Please double check that this is correct."
                },
                {
                  "label": "Running", 
                  "value": 5, 
                  "previous_value": 4, 
                  "previous_value_date": "2020-04-29", 
                  "required": true, 
                  "regression_warning_text": "Your pain score worsened (last submission: 4).", 
                  "regression_warning_subtext": "Please double check that this is correct."
                }, 
                {
                  "label": "Lifting toddler", 
                  "value": null, 
                  "previous_value": 3, 
                  "previous_value_date": "2020-04-29", 
                  "required": true, 
                  "regression_warning_text": "Your pain score worsened (last submission: 3).", 
                  "regression_warning_subtext": "Please double check that this is correct."
                }
              ]
            }
          }
        ], 
        "previous_url": "https://example.com/v2/progress-form-drafts/<draft-id>?section=1", 
        "next_url": "https://example.com/v2/progress-form-drafts/<draft-id>?section=3"
      }
    }


## Save Aggravating Activities Section Draft

Request endpoint is current URL.

- `PUT "https://example.com/v2/progress-form-drafts/12345?section=2"`

Request object sends the Component ID and the user-selected value.

    {
      "draft_values": [
        {
          "component_id": "d5b720f4-b7e8-406b-876a-73787c603622",
          "value": [2,3,4]
        }
      ]
    }

Response should be enough with the OK or CREATED status code.


## Get Functional Limitations Stiffness Sub Section Components

**Section 3**
Components:

- Header
- Sub header
- Text Radio Input

Request using section 2  `next_url` value.

- `GET "https://example.com/v2/progress-form-drafts/<draft-id>?section=3`

Response object

    {
      "draft": {
        "section": 3, 
        "sections_total": 5, 
        "components": [
          {
            "id": "8dbb97d8-a922-41eb-9a14-600695f46c9a", 
            "type": "header", 
            "data": {
              "label": "Functional Limitations", 
              "body": "This information will help us keep track of how you feel about your knee and how well you are able to do your usual activities. If you are unsure about how to answer a question, please give the best answer you can."
            }
          }, 
          {
            "id": "127d1d46-b8e4-4fdc-8cac-dfa60b310a8b", 
            "type": "subheader", 
            "data": {
              "label": "Stiffness",
              "body": "The following question concerns the amount of joint stiffness you have experienced during the *last week* in your knee. Stiffness is a sensation of restriction or slowness in the ease with which you move your knee joint."
            }
          },
          {
            "id": "d6b9690c-796a-4352-a99f-6aa25ede637e", 
            "type": "text_radio_input", 
            "data": {
              "question_text": "How severe is your knee stiffness after first waking up in the morning?", 
              "required": true, 
              "options": [
                {
                  "label": "None", 
                  "id": 1
                }, 
                {
                  "label": "Mild", 
                  "id": 2
                }, 
                {
                  "label": "Moderate", 
                  "id": 3
                }, 
                {
                  "label": "Severe", 
                  "id": 4
                }, 
                {
                  "label": "Extreme", 
                  "id": 5
                }
              ], 
              "value": "null"
            }
          }
        ],
        "previous_url": "https://example.com/v2/progress-form-drafts/12345?section=2", 
        "next_url": "https://example.com/v2/progress-form-drafts/12345?section=4"
      }
    }


## Save Functional Limitations Stiffness Sub Section Draft

Request endpoint is current URL.

- `PUT "https://example.com/v2/progress-form-drafts/12345?section=3"`

Request object sends the Component ID and the user-selected value.

    {
      "draft_values": [
        {
          "component_id": "d6b9690c-796a-4352-a99f-6aa25ede637e",
          "value": 3 // option choice ID
        }
      ]
    }

Response should be enough with the OK or CREATED status code.


## Get Functional Limitations Pain Sub Section Components

**Section 4**
Components:

- Header
- Sub header
- Text Radio Input

Request using section 3  `next_url` value.

- `GET "https://example.com/v2/progress-form-drafts/<draft-id>?section=4`

Response object.
This section has 4 questions. Just two added for short.

    {
      "draft": {
        "section": 4, 
        "sections_total": 5, 
        "components": [
          {
            "id": "8dbb97d8-a922-41eb-9a14-600695f46c9a", 
            "type": "header", 
            "data": {
              "label": "Functional Limitations", 
              "body": "This information will help us keep track of how you feel about your knee and how well you are able to do your usual activities. If you are unsure about how to answer a question, please give the best answer you can."
            }
          }, 
          {
            "id": "127d1d46-b8e4-4fdc-8cac-dfa60b310a8b", 
            "type": "subheader", 
            "data": {
              "label": "Pain", 
              "body": "What amount of knee pain have you experienced the last week during the following activities?"
            }
          },
          {
            "id": "d6b9690c-796a-4352-a99f-6aa25ede637e", 
            "type": "text_radio_input", 
            "data": {
              "question_text": "Twisting/pivoting on your knee ", 
              "required": true, 
              "options": [
                {
                  "label": "None", 
                  "id": 1
                }, 
                {
                  "label": "Mild", 
                  "id": 2
                }, 
                {
                  "label": "Moderate", 
                  "id": 3
                }, 
                {
                  "label": "Severe", 
                  "id": 4
                }, 
                {
                  "label": "Extreme", 
                  "id": 5
                }
              ], 
              "value": "null"
            },
            {
            "id": "d6b9690c-796a-4352-a99f-6aa25ede637e", 
            "type": "text_radio_input", 
            "data": {
              "question_text": "Straightening knee fully",
              "required": true, 
              "options": [
                {
                  "label": "None", 
                  "id": 1
                }, 
                {
                  "label": "Mild", 
                  "id": 2
                }, 
                {
                  "label": "Moderate", 
                  "id": 3
                }, 
                {
                  "label": "Severe", 
                  "id": 4
                }, 
                {
                  "label": "Extreme", 
                  "id": 5
                }
              ], 
              "value": "null"
            }
          }
        ],
        "previous_url": "https://example.com/v2/progress-form-drafts/12345?section=3", 
        "next_url": "https://example.com/v2/progress-form-drafts/12345?section=5"
      }
    }


## Save Functional Limitations Pain Sub Section Draft

Request endpoint is current URL.

- `PUT "https://example.com/v2/progress-form-drafts/12345?section=4"`

Request object sends the Component ID and the user-selected value.

    {
      "draft_values": [
        {
          "component_id": "d6b9690c-796a-4352-a99f-6aa25ede637e",
          "value": 3 // option choice ID
        },
        {
          "component_id": "d6b9690c-796a-4352-a99f-6aa25ede6378",
          "value": 2 // option choice ID
        }
      ]
    }

Response should be enough with the OK or CREATED status code.


## Get Functional Limitations Function Sub Section Components

**Section 5**
Components:

- Header
- Sub header
- Text Radio Input
- Submit Input

Request using section 4  `next_url` value.

- `GET "https://example.com/v2/progress-form-drafts/<draft-id>?section=5`

Response object.
This section has 2 questions. Added 1 for short.

    {
      "draft": {
        "section": 5, 
        "sections_total": 5, 
        "components": [
          {
            "id": "a150048c-76d5-4852-b7f8-890d91b3aa59", 
            "type": "header", 
            "data": {
              "label": "Functional Limitations", 
              "body": "The following questions concern your physical function. By this we mean your ability to move around and to look after yourself. For each of the following activities please indicate the degree of difficulty you have experienced in the *last week* due to your knee."
            }
          }, 
          {
            "id": "7385265c-4c28-45a8-8a5b-12d6db032ecd", 
            "type": "subheader", 
            "data": {
              "label": "Function, daily living", 
              "body": "The following questions concern your physical function. By this we mean your ability to move around and to look after yourself. For each of the following activities please indicate the degree of difficulty you have experienced in the last week due to your knee."
            }
          }, 
          {
            "id": "efb40e3c-fa51-4721-93ae-cd2bb1fbde16", 
            "type": "text_radio_input", 
            "data": {
              "question_text": "Rising from sitting", 
              "required": true, 
              "options": [
                {
                  "label": "None", 
                  "id": 0
                }, 
                {
                  "label": "Mild", 
                  "id": 1
                }, 
                {
                  "label": "Moderate", 
                  "id": 2
                }, 
                {
                  "label": "Severe", 
                  "id": 3
                }, 
                {
                  "label": "Extreme", 
                  "id": 4
                }
              ], 
              "value": 1
            }
          },  
          {
            "id": "01cfab27-c012-435f-b1ad-64a34b414096", 
            "type": "submit_button_input", 
            "data": {
              "label": "Submit"
            }
          }
        ], 
        "previous_url": "https://example.com/v2/progress-form-drafts/12345?section=4", 
        "next_url": "https://example.com/v2/progress-form-drafts/12345/submit"
      }
    }


## Save Functional Limitations Function Sub Section Draft

Request endpoint is current URL.

- `PUT "https://example.com/v2/progress-form-drafts/12345?section=4"`

Request object sends the Component ID and the user-selected value.

    {
      "draft_values": [
        {
          "component_id": "efb40e3c-fa51-4721-93ae-cd2bb1fbde16",
          "value": 3 // option choice ID
        },
        {
          "component_id": "efb40e3c-fa51-4721-93ae-cd2bb1fbab34",
          "value": 1 // option choice ID
        }
      ]
    }

Response should be enough with the OK or CREATED status code.

## Submit Progress Form

Request endpoint is `next_url`.

- `POST "https://example.com/v2/progress-form-drafts/12345/submit"`


# PUT Requests Objects per Sections
    // PAIN SCALE
    
    {
            "draft_values": [
                    {
                            "component_id": "f4baec36-186f-42ff-9e87-be7d7a9bf056",
                            "value": 2
                    }
            ]
    }
    
    // AGGRAVATING ACTIVITIES
    
    {
        "draft_values": [
            {
                "component_id": "f4baec36-186f-42ff-9e87-be7d7a9bf056",
                "value": [
                    2,
                    3,
                    7
                ]
            }
        ]
    }
    
    // FUNCTIONAL LIMITATIONS
    
    {
        "draft_values": [
            {
                "component_id": 58, // ID question
                "value": 172 // ID option choice
            }
    
            // OPTIONAL
            ,
            {
                "component_id": 58,
                "value": 172
            },
            {
                "component_id": 58,
                "value": 172
            }
        ]
    }

