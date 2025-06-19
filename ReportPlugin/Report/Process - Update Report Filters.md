# Update Report Options
This is documentation for the process file `process/process_update_report_filters.php`

The initialised variables returning null is used as a check to determine whether the report record on `report_list` gets updated and which fields either get updated or cleared.
Note that the variable `$report_id` cannot return null, if it some how does then the process will simply end without making changes.
```
// Get POST data
$report_id = isset($_POST['report_id']) ? intval($_POST['report_id']) : null;
$filters = isset($_POST['filters']) ? stripslashes($_POST['filters']) : null;
$comparison_type = isset($_POST['comparison_type']) ? sanitize_text_field($_POST['comparison_type']) : null;
$comparison_value = isset($_POST['comparison']) ? sanitize_text_field($_POST['comparison']) : null;
$report_title = isset($_POST['report_title']) ? sanitize_text_field($_POST['report_title']) : null;
$resident_title = isset($_POST['resident_title']) ? sanitize_text_field($_POST['resident_title']) : null;
$family_title = isset($_POST['family_title']) ? sanitize_text_field($_POST['family_title']) : null;
$report_logo = isset($_POST['report_logo']) ? esc_url_raw($_POST['report_logo']) : null;
```


The fields `$update_fields` are dependant on whether the previously mentioned variables return any values, and the variable `$formats` follow the correct data types corresponding with the returned `$update_fields`
```
$update_fields = [];
$formats = [];

// Include only fields that are set to avoid overwriting with null values
if (!is_null($filters)) {
    $update_fields['filters'] = $filters;
    $formats[] = "%s";
}
if (!is_null($comparison_json)) {
    $update_fields['comparison'] = $comparison_json;
    $formats[] = "%s";
}else{
    $update_fields['comparison'] = "{}";
    $formats[] = "%s";
}
if (!is_null($report_title)) {
    $update_fields['report_title'] = $report_title;
    $formats[] = "%s";
}
if (!is_null($resident_title)) {
    $update_fields['resident_name'] = $resident_title;
    $formats[] = "%s";
}
if (!is_null($family_title)) {
    $update_fields['family_name'] = $family_title;
    $formats[] = "%s";
}
if (!is_null($report_logo)) {
    $update_fields['logo_url'] = $report_logo;
    $formats[] = "%s";
}

// If no valid data is provided, return an error
if (empty($update_fields)) {
    wp_send_json_error(['message' => 'No valid data provided to update.']);
}
```

As you can see the update query is dynamic. The `$updated` variable should not attempt to update fields that return null for the available fields in the corresponding record.
```
// Perform the update
$updated = $wpdb->update(
    "report_list",
    $update_fields,
    ["id" => $report_id],
    $formats,
    ["%d"]
);
```
