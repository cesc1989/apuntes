# ✅ [SOLVED] Case: Order of Questions
In *ASES forms*, the first question of the Web Form is “*Sleep on your painful side”.* However, in Mobile Form the first question returned is “*Put on a coat”*. This is clearly an issue which could have very bad repercussions if the order of the questions in web isn’t matching the order of the answers are returned after being saved in Mobile form.

This document will navigate the issue to try to clear it out and find what’s the matter.

## Development Environment

First thing I found wrong is that in development environment I’ve been working with the wrong order of questions.

The rake task that creates questions for ASES forms defines the first one as “*Put on a coat”:*

    Question.find_or_create_by(form_type_id: ases.id, content: 'Put on a coat', order: 1)

But in staging and production the order value given to this question is **10**.

    // STAGING
     [20, 10, "Put on a coat"],
     [21, 1, "Sleep on your painful side"],
     [22, 2, "Wash back or do up bra in back"],
     [23, 3, "Manage toileting"],
     [24, 4, "Comb hair"],
     [25, 5, "Reach a high shelf"],
     [26, 6, "Lift 10lb above the shoulder"],
     [27, 7, "Do usual work"],
     [28, 11, "Do usual sports"],
     [29, 8, "Throw a ball overhead"],
     [30, 9, "Pain Scale"]
    
    // PRODUCTION
     [20, 10, "Put on a coat"],
     [21, 1, "Sleep on your painful side"],
     [22, 2, "Wash back or do up bra in back"],
     [23, 3, "Manage toileting"],
     [24, 4, "Comb hair"],
     [25, 5, "Reach a high shelf"],
     [26, 6, "Lift 10lb above the shoulder"],
     [27, 7, "Do usual work"],
     [28, 11, "Do usual sports"],
     [29, 8, "Throw a ball overhead"],
     [30, 9, "Pain Scale"]

Meaning that the order of the questions was updated directly through Rails console and never with a rake task that could be tracked by Git.

~~**TODO: Update the rake task to reflect to right order for ASES questions.**~~

The order of the questions doesn’t match the order described [in this doc](https://docs.google.com/spreadsheets/d/10LKyXlBv80BnpfCfULyYPLKPHiROwJ9zNzdAhfGWq6g/edit#gid=1646745280). **Do we need to ask Palak about this situation?**

# Why the order in Mobile form is different from Web form?

The endpoint `{{BASE_URL}}/form_types/:acronym/questions` where acronym is ASES returns the as the first question:

    {
        "questions": [
            {
                "id": 20,
                "content": "Put on a coat",
                "form_type_acronym": "ases",
                "order": 10,
            },

In the mobile API, the first question in the JSON is *“Put on a coat”* as well:

    {
        "draft": {
            "category_id": 3,
            "components": [
                {},
                {
                    "id": "21",
                    "data": {
                        "question_text": "Put on a coat",
                        "selected_option_id": null,
                        "options": []
                }

So if it’s the first in both responses Why is it shown as the *ninth* question in Web form?

> First suspicion is that the order is handled in frontend code. But I couldn’t find anything that alters the order of the questions after the JSON response.

~~**@nicolas TODO: Is the order of questions handled by frontend code?**~~

Yes, it is. The list of questions returned in the `/questions` endpoint is ordered by frontend client using the `order` attribute of each element.

~~**TODO: Return list of questions ordered by the**~~ `~~**order**~~` ~~**attribute instead of creation date.**~~

After updating the endpoint to return the questions ordered by the `order` attribute, the “*Put on a coat”* question isn’t returned anymore as the first one in the JSON response. Also tried a form in development and looks OK.

~~**@nicolas TODO: Why is the question “Pain Scale” returned as the last one? Why is the “Put on a coat” question returned as the ninth one when its**~~ `~~**order**~~` ~~**value was 1?**~~

The *“Pain Scale”* question is taken out of the *questions* array and moved to the end of the Functional Limitations section.

The *“Put on a coat”* question has order 10 but when the *“Pain Scale”* question is removed, its order value goes down by 1 and it’s displayed as question #9.

# Worst Part of the Issue

The problem isn’t that the questions is in a bad order **but** when we open the same form in Mobile client and Web client, the order changes and the answers don’t match as shown in the image:

![This is the form in question.](https://paper-attachments.dropbox.com/s_02C1123C6FEEC245CF397CDC4A245B79C00394DE66A44146BAB0F209F1BE2C33_1597684663862_image.png)


Question *“Put on a coat”* appears as the ninth in Web form and as first in Mobile form with a different answered choice.

## Web Form

This error doesn’t happen in Web form. Whatever the order I save answers in a web form, the option I chose for a given question appears always in the correct spot.

> Tried with [this](https://forms.luna.farzoo.click/patients/555/forms/f1bad7f6-1505-5b3a-cf53-9f7bf170c17e) form.

I tried saving answers using Postman and the options appeared in the expected questions and choices.

## Mobile Progress Form

Tried updating answers for [this](https://forms.luna.farzoo.click/patients/f366e35e-7342-4c03-a25c-d5f53b409df8/forms/98625953-10b9-4c82-a81c-ad199424f6cc) form using V2 API (through Postman) and when reviewed it in the Web version, it was fine.

Reset the form and save all answers using the V2 endpoint (Postman again) AND **this is where it breaks.**

The question and its choices are saved correctly but when being displayed in a web form, they don’t match.

I think this is happening because the render of questions, when there’s a draft, is tightly couple to the JSON response of the `answers_attributes` attribute.

Look at this image:

![](https://paper-attachments.dropbox.com/s_02C1123C6FEEC245CF397CDC4A245B79C00394DE66A44146BAB0F209F1BE2C33_1597765708462_000-mobile-breaks-web.png)


The first saved answer is for question *“Put on a coat”* with **ID 20 and choice ID 38** (Very Difficult).

In the image we can see that the JSON value for the choice is the first one in the `answers_attributes` array BUT the *“Put on a coat”* question is being assigned the value of ANOTHER answer.

Looks like the order of the answers commands the order of the questions.

**This only occurs when saving progress in a Progress Mobile form and then opening in the Web form.**

**Fixed by** https://github.com/lunacare/patient-forms-frontend/pull/327 👏🏼 


