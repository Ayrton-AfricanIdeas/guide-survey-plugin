# Report Builder - Comment data
This is documentation for the process file `process\process-save-comments-form-builder.php`

The first check on this process is to make sure that there are comment data to update or record. As simple as the conditional check is, it is important to note that if there is no comment data to update or record, we have to clear all comment data in the `report_survey_comments` table where the fields `report_id` and `cq_uid` in the table `report_survey_comments` matches the values of `$report_id` and `$q_uid` respectively in the function `delete_all_comments` found at the bottom of the process.
```
// Ensure `commentsData` is set
if (!isset($_POST['commentsData']) || !is_array($_POST['commentsData'])) {
    if(isset($_POST['question_order']) && isset($_POST['report_id'])){
        $delete_this_question_comments = delete_all_comments(get_q_uid(get_form_id($_POST['report_id']), $_POST['question_order']), $_POST['report_id']);
        if($delete_this_question_comments){
            wp_send_json_success("Records processed successfully.");
            wp_die();
        }
    }
    
    wp_send_json_error("No data provided.");
    wp_die();
}
...
function delete_all_comments($q_uid, $report_id){
    global $wpdb;
    
    $query = "DELETE FROM `report_survey_comments` WHERE `cq_uid` = %s AND `report_id` = %d";

    $results = $wpdb->query($wpdb->prepare($query, $q_uid, $report_id));
    return $results;
    
}
```

Note that these variables get initialised after the above `commentsData` check as we wouldn't need to do any other processes if the `commentsData` is empty.
These variables are typically the only data we need. Everything else that needs to be queried can come from the functions created at the bottom of the process.
```
$commentsData = $_POST['commentsData'];
$form_id = sanitize_text_field($commentsData[0]['form_id']); // Sanitize input (it's a string)

...
function to_bit($value) {
    return ($value === true || $value === 'true' || $value === 1 || $value === '1') ? 1 : 0;
}

function get_form_id($report_id){
    global $wpdb;
    $query = "SELECT `survey_code` FROM `report_list` WHERE `id` = %d";
    $result = $wpdb->prepare($query, $report_id);
    return $wpdb->get_var($result);
}
function get_q_uid($form_id, $question_order){

    global $wpdb;

    $query = "SELECT `q_uid` FROM `question_list` WHERE `form_id` = %s AND `question_order` = %d";
    $result = $wpdb->prepare($query, $form_id, $question_order);
    return $wpdb->get_var($result);
}
function delete_all_comments($q_uid, $report_id){
    global $wpdb;
    
    $query = "DELETE FROM `report_survey_comments` WHERE `cq_uid` = %s AND `report_id` = %d";

    $results = $wpdb->query($wpdb->prepare($query, $q_uid, $report_id));
    return $results;
    
}
```

The next processes that follow is done in a specific order to ensure we know whether records need to be deleted, inserted, or updated. It is important that the steps are followed in the manner they are layed out.
We want to find the comments that are not updated or inserted, any records left over should be found by their `id` field in the `report_survey_comments` table and be removed.
Go through each step in the process for further explanation. 
The steps are indicated by comments like this:
```
// Step 1: Fetch existing records for the specified form_id
$existingRecords = $wpdb->get_results($wpdb->prepare(
    "SELECT id, report_id, form_id, survey_id, cq_uid, question_order, comment, sentiment, edit_comment, include_comment 
     FROM report_survey_comments 
     WHERE form_id = %s",
    $form_id
), ARRAY_A);
```
