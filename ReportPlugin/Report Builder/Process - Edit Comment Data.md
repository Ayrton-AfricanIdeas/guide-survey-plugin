# Report Builder - Comment data

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
