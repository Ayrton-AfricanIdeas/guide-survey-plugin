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
Within the PHP function `filter_layout` it generates a javascript function within a script tag called `populateStoredFilters`. On load, this function executes and retrieves potential filters already stored by the user for the survey responses they are viewing. The process `fn_get_stored_filters` executes when the ajax request within the function `populateStoredFilters` runs sends the retrieved potential filters to the frontend to populate the generated filters accordinly. Please review the process `fn_get_stored_filters` function in `survey_plugin.php`.
Once the ajax request returns succesfully the potential filters will populate the fields in the filter layout.

#### Store Selected Filters
Within the PHP function `filter_layout`, the 
```
$('#applyFilter').on('click', function() {
    applyFilter();
});
```
