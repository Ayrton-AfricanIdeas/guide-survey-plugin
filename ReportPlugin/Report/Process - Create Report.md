# Create Report

Initialising the variables `$current_user_id` is only really needed to get the `company_id` which is a user meta field to identify the company a particular user is under. Only users under the subscriber role are under a company_id.
We can cross reference the company_id against the unique field `id` to get the value of the field `name` of that record from the table `cr_companies`. But typically, we really only need the `company_id`.
```
global $wpdb;
$current_user_id = get_current_user_id();
$company_id = get_user_meta($current_user_id, 'company_id', true);
```
This process is quite simple. The insert query is simply pushed to the table `report_list`. `$title` and `$survey_code` variables are extracted from `$_POST['report_data']` that gets posted via ajax, see (reserved space: refactored js file)

```
$report_data = json_decode(stripslashes($_POST['report_data']), true);

// Extract & Sanitize Data
$title = isset($report_data['title']) ? sanitize_text_field($report_data['title']) : '';
$survey_code = isset($report_data['survey']) ? sanitize_text_field($report_data['survey']) : '';

if (empty($title) || empty($survey_code)) {
    wp_send_json_error(['message' => 'Title and survey are required.']);
    return;
}
// Insert Report with Required Fields
$inserted = $wpdb->insert("report_list", [
    "user_id"      => $current_user_id,
    "company_id"   => $company_id,
    "report_title" => $title,
    "survey_code"  => $survey_code
]);
```
