---
kind: letter
title: Engagement letter
respondent_type: entity
jurisdiction: NV
confidential: true
questionnaire:
  BEGIN:
    _: END
  END: {}
workflow:
  BEGIN:
    intake_submitted: lawyer_review
  lawyer_review:
    approved: END
    rejected: END
  END: {}
---

Replace this placeholder with the notation this Project actually uses.
