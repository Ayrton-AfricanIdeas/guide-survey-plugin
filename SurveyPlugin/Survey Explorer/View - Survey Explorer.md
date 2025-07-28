# Survey Explorer
This section allows you to view responses for a specific survey and its variants. You can explore individual survey responses, see a summary of specific questions, and apply filters to narrow down the results.
The functions used within this section are mainly used to display either survey responses or different types of summaries. The summaries several types of summaries, each type of question, and each have their own function which we'll go into a bit more detail later. I will not be able to fully explain each function in detail without sharing the entire code, so it might be best to review this file within the plugin as we go through each section.

## Filter
<img width="1722" height="494" alt="image" src="https://github.com/user-attachments/assets/4bd3606c-4b37-4bf7-9c23-db78f6fcdac6" />

When the "Apply Filters" button is selected, the chosen filters are saved to the `filtered_survey_filters` table for the corresponding survey. This allows the filters to persist for each user, since the user ID is stored alongside the filters, the same selections will be automatically restored the next time that user accesses the survey.
The process that stores these filters is called `fn_filter_survey_responses` and the process is under the function of the same name within the `survey_plugin.php` file.
The function used to generate the filter is called `filter_layout`.
### Process Filter Survey
#### On Load - Populate Stored Filters
Within the PHP function `filter_layout` it generates a javascript function within a script tag called `populateStoredFilters`. On load, this function executes and retrieves potential filters already stored by the user for the survey responses they are viewing. The process `fn__stored_filters` executes when the ajax request within the function `populateStoredFilters` runs sends the retrieved potential filters to the frontend to populate the generated filters accordinly. Please review the process `fn_get_stored_filters` function in `survey_plugin.php`.
Once the ajax request returns succesfully the potential filters will populate the fields in the filter layout.

#### Store Selected Filters
Within the PHP function `filter_layout`, the javascript function `applyFilter` is generated and gets executes once the button with the id `#applyFilter` is clicked.
```
$('#applyFilter').on('click', function() {
    applyFilter();
});
```
An AJAX request triggers the `fn_filter_survey_responses` process. The function that executes this process is defined in `survey_plugin.php` and shares the same name. The selected filter options on the frontend are passed to the process via the AJAX request, which then either updates, inserts, or deletes a record in the `filtered_survey_filters` table. This action is based on a match between the user ID of the person selecting the filters and the survey ID they are viewing.
Once the AJAX request completes successfully, the page reloads.

#### Clear Selected Filters
The PHP function `filter_layout` generates a JavaScript function that deletes a record from the `filtered_survey_filters` table, based on a match between the user ID and the survey ID. This is done to clear any previously applied filters.

The button with the ID `clearFilter` triggers an AJAX request that executes the `fn_clear_survey_filters process`, which simply deletes the corresponding record. Once the AJAX request completes successfully, the page reloads. This process is defined in `survey_plugin.php`.
```
// **Clear Filter Button**
$('#clearFilter').on('click', function() {
...
```
Note: There's no particular reason why `applyFilter()` has its own named function while the "Clear Selected Filters" logic is defined inline within the click event. Once we are ready to refactor the plugin, we can revisit this decision and determine the most appropriate structure. For now, this distinction shouldn't have any significant impact.

### Query Results
Querying the results goes through quite a few conditions depending on the responses on the survey. The main function that you should consider reviewing is called `get_filtered_responses`. The second function would be `get_responses`, which is simply the queried results if there are no filters in place for the survey that is being viewed.

```
// Use correct table for filters
$filter_table = "filtered_survey_filters";

// Retrieve all stored filters for this user and form
$stored_filters = $wpdb->get_results($wpdb->prepare(
    "SELECT question_id, selected_answers FROM $filter_table WHERE user_id = %d AND form_code = %s",
    $user_id, $form_code
));

// Convert results into an array format
$filters = [];
foreach ($stored_filters as $filter) {
    $filters[$filter->question_id] = maybe_unserialize($filter->selected_answers);
}

// Apply filters if they exist
if (!empty($filters)) {
    $responses = get_filtered_responses($form_code, $filters);
} else {
    // No stored filters, load full survey responses
    if($view == 'responses'){
        $responses = get_responses($form_code, $show_all ? null : $limit);
    }else{
        $responses = get_responses($form_code, $show_all ? null : $limit, true);
    }
    
}
```
Once the results are queried and initialised as the variable `$responses` get looped and organised into an array called `$survey` to be called later when displaying the survey responses or summaries.

```
// Organize responses by survey_id
$surveys = [];
foreach ($responses as $response) {
    $surveys[$response->survey_id]['questions'][] = [
        'survey_id' => $response->survey_id,
        'question_id' => $response->question_id,
        'question_value' => $response->question_value,
        'question_type' => $response->question_type,
        'options' => maybe_unserialize($response->options),
        'answer' => maybe_unserialize($response->answer),
        'date_created' => $response->date_created
    ];
}
```
