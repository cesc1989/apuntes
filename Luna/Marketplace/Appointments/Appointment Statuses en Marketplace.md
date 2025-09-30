# Appointment Statuses en Marketplace

Etiquetas: #luna_help_desk 

Definidos por la clase AppointmentStatus en `app/marketplace/constants.py`.

```python
class AppointmentStatus:
    # ----- Default -----
    SCHEDULED = "scheduled"

    # ----- Session actions -----
    THERAPIST_STARTED = "in_progress"
    THERAPIST_FINISHED = "finished"

    # ----- Cancellation -----
    CANCELLED_BY_ADMIN = "cancelled_by_admin"
    CANCELLED_BY_PATIENT = "cancelled_by_patient"
    CANCELLED_BY_PATIENT_SHORT_NOTICE = "cancelled_by_patient_short_notice"
    CANCELLED_BY_THERAPIST = "cancelled_by_therapist"
    CANCELLED_BY_THERAPIST_SHORT_NOTICE = "cancelled_by_therapist_short_notice"

    # ----- No-Show -----
    NO_SHOW_PATIENT = "no_show_by_patient"
    NO_SHOW_THERAPIST = "no_show_by_therapist"

    # ----- Removed (Internal)-----
    REMOVED = "removed"

    @classmethod
    def all(cls):
        return {
            cls.SCHEDULED,
            cls.THERAPIST_STARTED,
            cls.THERAPIST_FINISHED,
            cls.CANCELLED_BY_ADMIN,
            cls.CANCELLED_BY_PATIENT,
            cls.CANCELLED_BY_PATIENT_SHORT_NOTICE,
            cls.CANCELLED_BY_THERAPIST,
            cls.CANCELLED_BY_THERAPIST_SHORT_NOTICE,
            cls.NO_SHOW_PATIENT,
            cls.NO_SHOW_THERAPIST,
            cls.REMOVED,
        }

    @classmethod
    def billable_types(cls):
        return {cls.THERAPIST_STARTED, cls.THERAPIST_FINISHED, cls.CANCELLED_BY_PATIENT_SHORT_NOTICE, cls.NO_SHOW_PATIENT}

    @classmethod
    def default(cls):
        return cls.SCHEDULED

    @classmethod
    def cancellation_types(cls):
        return {
            cls.CANCELLED_BY_ADMIN,
            cls.CANCELLED_BY_PATIENT,
            cls.CANCELLED_BY_PATIENT_SHORT_NOTICE,
            cls.CANCELLED_BY_THERAPIST,
            cls.CANCELLED_BY_THERAPIST_SHORT_NOTICE,
            cls.NO_SHOW_PATIENT,
            cls.NO_SHOW_THERAPIST,
            cls.REMOVED,
        }
```