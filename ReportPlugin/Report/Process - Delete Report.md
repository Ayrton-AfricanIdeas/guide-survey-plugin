# Delete Report
This is documentation for the process file `process/process_delete_report.php`

Before clearing the report record from the table `report_list`, the edited comment data see (reserve space for report builder guide)
We do only need the id of the report to clear any relevant data retrived from `$_POST['report_id']` sent via ajax, see (reserve space for js file after refactor).
```
function delete_comment_data($id){
    global $wpdb;
    $result = $wpdb->delete('report_survey_comments', ['report_id' => $id], ['%d']);
    return $result;
}
function delete_report($id){
    global $wpdb;
    $result = $wpdb->delete('report_list', ['id' => $id], ['%d']);
    return $result;
}
```
