# Survey Explorer
This section allows you to view responses for a specific survey and its variants. You can explore individual survey responses, see a summary of specific questions, and apply filters to narrow down the results.

## Filter
<img width="1722" height="494" alt="image" src="https://github.com/user-attachments/assets/4bd3606c-4b37-4bf7-9c23-db78f6fcdac6" />

When the "Apply Filters" button is selected, the chosen filters are saved to the `filtered_survey_filters` table for the corresponding survey. This allows the filters to persist for each user, since the user ID is stored alongside the filters, the same selections will be automatically restored the next time that user accesses the survey.
The process that stores these filters is called `fn_filter_survey_responses` and the process is under the function of the same name within the `survey_plugin.php` file.
