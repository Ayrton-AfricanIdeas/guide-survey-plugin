# Record Response
This process is found in the file `process/process_response.php`.

For each question answered in a survey this process will run to attempt to insert a new record. The reason for this is Care Research specifically wants responses to be recorded as a user is answering a survey and not at the end of the survey, so whenever a user clicks "next" to answer the next question the previous response has already been recorded as a response.

### Initialise Variables

The initial variables that get initialised are passed through from an ajax request from the file `js/form-response.js`. These variables have their own field in where they get inserted as a record in the `survey_responses` table.


| Variable      | Field      |
| ------------- | ------------- |
| `$form_id` | `form_id` |
| `$survey_id` | `survey_id` |
| `$question_id` | `question_id` |
| `$response`  | The `response` variable must be handled differently dependant on the `$response['type']` value, the value is the extracted and stored in a variable `$answer` and later inserted in the field `answer` |
| `$is_variant` | The relevant field that gets inserted is `variant` which is a bit field |
| `$original_form_code` | `original_form_code`. This field only gets recorded if `$is_variant` returns true. |
| `$q_uid` | `rq_uid` |
