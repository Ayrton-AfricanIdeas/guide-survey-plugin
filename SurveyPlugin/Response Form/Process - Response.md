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

After the above variables get initialised, we have to ensure the important values posted via the ajax request exists or is valid, if not we simply end the process without inserting any records and return an error message to the frontend.
```
if (!$form_id || !$survey_id || !$q_uid || !$response) {
    echo json_encode(['error' => 'Missing required data']);
    wp_die();
}
if (get_survey_status($is_variant, $form_id) != 'open') {
    echo json_encode(['error' => 'Survey is closed']);
    wp_die();
}

function get_survey_status($is_variant, $form_code) {
    global $wpdb;
    
    // Define table based on whether it's a variant
    $table = $is_variant ? 'survey_variant_types' : 'form_list';
    
    // Prepare query
    if($is_variant){
        $query = "SELECT `status` FROM `$table` WHERE form_code = %s or variant_form_code = %s";
        $result = $wpdb->get_row($wpdb->prepare($query, $form_code, $form_code));

    }else{
        $query = "SELECT `status` FROM `$table` WHERE form_code = %s";
        $result = $wpdb->get_row($wpdb->prepare($query, $form_code));

    }
    // error_log('is variant sql: '.$wpdb->prepare($query, $form_code, $form_code));
    
    // Return status if found, otherwise return 'closed' (or any default value)
    return $result ? $result->status : 'closed';
}
```
### Update Response
The order of operations in this process is that we should first check if there is a response record to update, otherwise we simply create a new response record.
It would be useful to familiarise yourself with the table structure of the `survey_responses` table. The structure is explained `SurveyPlugin/Table Structure/Table Structure - Responses.md`

Before performing an update, `$existing_record` is a variable initialised to check if there are any records that exist in the table
```
    // Check if the record exists
    $existing_record = $wpdb->get_var(
        $wpdb->prepare(
            "SELECT COUNT(*) FROM survey_responses WHERE survey_id = %s AND rq_uid = %s",
            $survey_id,
            $q_uid
        )
    );
```
