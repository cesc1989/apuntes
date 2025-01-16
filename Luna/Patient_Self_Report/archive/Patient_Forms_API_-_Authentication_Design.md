# Patient Forms API - Authentication Design
**Entities:**

- Patient Forms API
- Backend
- Marketplace

**Current State of Affairs:**

- Ruby backend *uses* Marketplace *uses* Patients Forms. Only Marketplace has access to Patients Forms.

**What we want to achieve:**

- Ruby backend *uses* Patients Forms (mobile-specific workflows)
- Marketplace *uses* Patients Forms (form status, appointment reminders, etc)

**Future wishlist:**

- Ruby backend *uses* Patients Forms (appointment reminders, mobile-specific workflows, form status)

Now that the Patient Forms API will be used by more than one client (currently is only used by Patient Forms Frontend client) a security layer becomes needed.

# Proposed Design
![](https://paper-attachments.dropbox.com/s_A4EE2D0955F01961FBEA1A6EE0A4F78145B86810E6A082556AF783617E5BCA4D_1579192635245_Patient+Forms+Token+Auth.png)


…

