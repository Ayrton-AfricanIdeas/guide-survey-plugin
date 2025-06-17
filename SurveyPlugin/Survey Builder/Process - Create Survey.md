# Create Survey
The following descriptions describe the process in which the process_create_survey.php generates surveys and questions.
Creating a survey involves two parts, the survey details which holds information on the survey itself within the table `form_list`, and the second part that holds records of the each question for the survey we create in a table named `question_list`.

Initialised variables:

| Variable              | Description                                                                                                                                                                                                                                              |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $user_id              | We mainly need the user_id to get the company_id of the user generating the survey. But there might be a future use for storing the user id as well.                                                                                                     |
| $forms_table_name     | This variable only stores the name of the table name `form_list` which gets used in queries within this process                                                                                                                                          |
| $questions_table_name | This variable only stores the name of the table name `question_list` which gets used in queries within this process                                                                                                                                      |
| $responses            | We need a way to send a message to the frontend once this process succeeds or fails. With the message we can debug the process, or a message could help us decide how we want to proceed forward on the frontend dependent on the result of the process. |
| $formDetails          | An array sent via an ajax request to store the survey details: title, description and evidence category                                                                                                                                                  |
| $dirty_form_data      | An array of questions sent via an ajax request. Note this data still needs to be processed.                                                                                                                                                              |

Generating the form code:
The below snippet generates a random 10 character string made up of numbers. When the code is generated, it is then checked against the table `form_list` against the field 'form_code' to see if there is any record that already uses the $code that we generated, if it does, the while continues since the number of records is not 0, else if the number of records that match $code is 0 then the loop simply breaks.

```
$code = '';
// Generate a unique form code
while (true) {
    $code = mt_rand(1000000000, 9999999999);
    $code = sprintf("%'.09d", $code);
    $results = $wpdb->get_results("SELECT 'form_list' AS source_table, form_code AS code
                                    FROM form_list
                                    WHERE form_code = '$code'

                                    UNION

                                    SELECT 'survey_variant_types' AS source_table, variant_form_code AS code
                                    FROM survey_variant_types
                                    WHERE variant_form_code = '$code';
                                    ");
    if (count($results) === 0) {
        break;
    }
}
```

Before handling any questions, we need to insert the survey details in the table `form_list` to establish the creation of the survey before adding questions.
```
// Insert form details into form_list table
$form_insert_result = $wpdb->insert(
    $forms_table_name,
    [
        'user_id' => $user_id,
        'company_id' => $formDetails['company_id'],
        'form_code' => $code,
        'title' => $formDetails['survey_title'],
        'description' => $formDetails['survey_desc'],
        'evidence_category' => $formDetails['evidence_category']
    ]
);
```

When inserting records into `question_list` table we need to generate a random unique id. Its necessary for referencing answers to a specific questions and responses in reporting and creating variant questions.
When a unique id is generated, we have to check at least within the survey that it is unique, this might be a bit redundant as the chances of it repeating is quite low.
```
// Prepare the insert query for questions
$insertQuery = "INSERT INTO `$questions_table_name` (form_id, question_order, question_value, question_type, options, key_question, quality_statement, comment_labels, core_question, core_question_id, q_uid) VALUES";

$used_question_uids = []; // to track q_uids within this batch

// Loop through each question and build the query
foreach ($dirty_form_data as $dirty_data) {
    $question_value = $dirty_data['question'];
    $question_type = $dirty_data['type'];
    $options_json = json_encode($dirty_data['options'], JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE);
    $question_order = $dirty_data['order'];
    $core_question = isset($dirty_data['core_question']) ? $dirty_data['core_question'] : 0;
    $core_question_id = isset($dirty_data['core_question_id'] ) ? $dirty_data['core_question_id'] : 0;
    // Initialize additional fields to null
    $key_question = $quality_statement = $comment_labels_json = 'NULL';

    // If the question type is "star-rating", extract additional fields
    if ($question_type === 'star-rating') {
        $key_question = isset($dirty_data['options']['key_question']) ? "'" . esc_sql($dirty_data['options']['key_question']) . "'" : 'NULL';
        $quality_statement = isset($dirty_data['options']['quality_statement']) ? "'" . esc_sql($dirty_data['options']['quality_statement']) . "'" : 'NULL';

        // Encode comment labels as JSON if they exist, with slashes and unicode unescaped
        if (isset($dirty_data['options']['comment_labels'])) {
            $comment_labels_json = "'" . esc_sql(json_encode($dirty_data['options']['comment_labels'], JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE)) . "'";
        }
    }

    //loop to make sure there are no duplicates in the survey.
    // This is redundant but there is a very low chance it can happen
    do {
        $q_uid = wp_generate_uuid4();
    } while (in_array($q_uid, $used_question_uids));
    $used_question_uids[] = $q_uid;
        // Append to the insert query, directly inserting the JSON fields without re-encoding

    $insertQuery .= sprintf(" ('%s', %d, '%s', '%s', '%s', %s, %s, %s, %d, %d, '%s'),",
        $code,
        $question_order,
        esc_sql($question_value),
        esc_sql($question_type),
        esc_sql($options_json),
        $key_question,
        $quality_statement,
        $comment_labels_json,
        $core_question,
        $core_question_id,
        esc_sql($q_uid)
    );
}
```

The function `check_conditional_radio_options` is used to apply the `q_uid` field of the questions selected by all branch questions in the survey after being inserted by taking the `question_order` selected, found in a json object stored in the `options` field of that branch question, and finding the `q_uid`.
```
function check_conditional_radio_options($form_id) {
    global $wpdb;

    // Get all conditional_radio questions for the form
    $query = "SELECT id, options FROM question_list WHERE form_id = %s AND question_type = 'conditional_radio'";
    $results = $wpdb->get_results($wpdb->prepare($query, $form_id));

    foreach ($results as $result) {
        $options = json_decode($result->options, true);
        $changed = false;

        if (!is_array($options)) {
            continue;
        }

        foreach ($options as $key => &$opt) {
            $branch = trim((string) ($opt['branch'] ?? ''));
            $order = trim((string) ($opt['order'] ?? ''));

            // Check if branch is missing or invalid
            if (empty($branch) || strtolower($branch) === 'null') {
                if ($order !== '') {
                    // Try to find q_uid by form_id and question_order
                    $q_uid = $wpdb->get_var(
                        $wpdb->prepare(
                            "SELECT q_uid FROM question_list WHERE form_id = %s AND question_order = %d LIMIT 1",
                            $form_id,
                            intval($order)
                        )
                    );

                    if ($q_uid) {
                        $opt['branch'] = $q_uid;
                        $changed = true;
                    }
                }
            }
        }

        // Update the options in DB if anything changed
        if ($changed) {
            $updated_options = wp_json_encode($options, JSON_UNESCAPED_UNICODE);
            $wpdb->update(
                'question_list',
                ['options' => $updated_options],
                ['id' => $result->id],
                ['%s'],
                ['%d']
            );

            error_log("Updated conditional_radio options for question ID {$result->id}");
        }
    }
}
```

