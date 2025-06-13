# Update Survey
The key thing to take note about most update processes and especially this `process_update_survey.php` is that unlike the `process_create_survey.php` it doesn't just query to do achieve a single process, which would be insert for the `process_create_survey.php`, the `process_update_survey.php`  performs all CRUD functions.

Taking a look at these variables that are initialised at the start of the process, they are all necessary in order to identify which records need to be inserted, updated, and/or deleted.
```
// Fetch existing question q_uids for this form
$existing_questions = $wpdb->get_results(
    $wpdb->prepare("SELECT id, q_uid FROM question_list WHERE form_id = %s", $form_code),
    ARRAY_A
);

$existing_uids = array_column($existing_questions, 'q_uid');
$incoming_uids = [];
$insert_count = 0;
$update_output = [];

// MAIN LOOP: Handle inserts and updates in order
$new_questions = [];
```

The below loop seperates the records that need to be inserted and generates a `q_uid` for each before inserting them in the `question_list` table and placing them in the `$new_questions` variable 1 record at a time.
The rest of the records would simply be updated if they are not new questions.
```
foreach ($fields_to_update as $index => $q) {
    $q_uid = isset($q['quid']) ? trim($q['quid']) : null;
    if ($q_uid === '' || $q_uid === 'null' || strtolower($q_uid) === 'null') {
        $q_uid = null;
    }
    
    $data = [
        'form_id' => $form_code,
        'question_order' => $index + 1,
        'question_value' => $q['question'],
        'question_type' => $q['type'],
        'conditional' => intval($q['conditional']),
        'conditional_id_page' => intval($q['conditional_id_page']),
        'required' => boolval($q['required']),
        'options' => json_encode($q['options'] ?? []),
        'key_question' => $q['options']['key_question'] ?? '',
        'evidence_category' => $q['options']['evidence_category'] ?? '',
        'quality_statement' => $q['options']['quality_statement'] ?? '',
        'comment_labels' => json_encode($q['options']['comment_labels'] ?? []),
        'core_question' => isset($q['core_question']) ? intval($q['core_question']) : 0,
        'core_question_id' => isset($q['core_question_id']) ? intval($q['core_question_id']) : 0
    ];
    

    if (empty($q_uid)) {
        // New question
        $q_uid = wp_generate_uuid4();
        $data['q_uid'] = $q_uid;
        $wpdb->insert('question_list', $data);
        $insert_count++;

        $new_questions[] = [
            'order' => $index + 1,
            'q_uid' => $q_uid
        ];
    } else {
        // Existing question
        $incoming_uids[] = $q_uid;
        $result = $wpdb->update('question_list', $data, ['q_uid' => $q_uid]);
        $update_output[] = $result !== false ? "Updated {$q_uid}" : "Failed {$q_uid}";
    }
}
```
From the `$existing_uids` variables we initialised earlier, we can find the `q_uid`s that were not updated, those questions are intended to be deleted as the do not appear in the array of questions sent from the frontend via ajax request.
```
// DELETE: Remove questions that are no longer in the update payload
$uids_to_delete = array_diff($existing_uids, $incoming_uids);
foreach ($uids_to_delete as $uid) {
    $wpdb->delete('question_list', ['q_uid' => $uid]);
}
```
