# Debugging Error modal after submitting Intake Form

# First error: “This form was already completed”

It's returned in the frontend when a requests to the submit endpoints respond with a 422 code.

![](https://paper-attachments.dropbox.com/s_A5AFB9D1ACB807C250D7F5A37C4DA7B0F5393DDD1418C86C7F73943F57B6137F_1638887566931_image.png)


**Why would an open form, being filled in, upon submission return 422?**

Forms in Prod where this happened

    c4aa6f01-cbdd-4d6d-bbbc-1980656a2ac7
    
    37f30e61-4632-4884-83e0-bb898ee82b40
> **NOTE**: all of latest forms are Quick Dash, and 2nd intake with accepted terms.

If a form is marked as completed:

- the draft won't save more data because there's a control in place for that
- when clicking submit the endpoint will respond with a 422 code

The "POST onboarding_forms" endpoint is configured to send the 422 error data to Sentry but we don't see that in Sentry.

## Suspicion #1 - Signature Generation

This *was* another issue *solved* by rescuing in the `SignatureGenerator` and `SignatureUploader` classes.

Caused by errors we see in Sentry for the "POST onboarding_forms" when something fails with Image Magick:

    MiniMagick::Error: `magick mogrify -font /usr/src/app/app/assets/fonts/Mistral.woff -gravity center

Tried forcing an error in the `SignatureGenerator` class and the response is a 500 error. The frontend in this case shows the message:

![](https://paper-attachments.dropbox.com/s_A5AFB9D1ACB807C250D7F5A37C4DA7B0F5393DDD1418C86C7F73943F57B6137F_1638889515108_image.png)


By rescuing in the `SignatureGenerator` and `SignatureUploader` classes we can provide a better experience for the user.

**Conclusion for Suspicion #1**
By rescuing the exceptions the form will complete its process as usual but if this fails the signature won’t be created.

We have to move this process to a background job to shorten the request cycle and also provide better handling for something beyond the actual form submission.

## Suspicion #2 - Missing Important Fields

Would a required empty value, left empty because of a server error, might be causing trouble to complete form?

We see in Suspicion #1 the signature might be missing when something goes wrong with Image Magick on the server. Could this be causing the error or a different error altogether?


# Second error: “An error occurred on the server. Try again in a few moments”

Got this error while submitting a local form

![](https://paper-attachments.dropboxusercontent.com/s_A5AFB9D1ACB807C250D7F5A37C4DA7B0F5393DDD1418C86C7F73943F57B6137F_1686064659708_image.png)


In this example, the error is triggered from the AdmissionForm class. It does something on the model and because of the error, the request never gets to the success path in the controller. It instead returns a 400 error code.

    method=POST path=/forms/10c020de-7fd5-43d4-a263-1730f8ccbf7b/onboarding_forms format=json controller=Api::V1::OnboardingFormsController action=create status=400 duration=1008.94 view=35.05 db=16.67 unpermitted_params=["form_id"...]

Confirmed this happens when trying to POST to the webhook URL.

![](https://paper-attachments.dropboxusercontent.com/s_A5AFB9D1ACB807C250D7F5A37C4DA7B0F5393DDD1418C86C7F73943F57B6137F_1686171081782_image.png)


See [error](https://sentry.alpha.getluna.com/share/issue/9f70393dde854fe899a493b02ba557cc/) in Sentry.

