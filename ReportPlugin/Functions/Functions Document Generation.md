# Document Generation Functions
The functions mentioned here can be found on the file `functions/functions_query_doc_gen.php`.

### Function get_rating_answers:
The queries used in reporting can be quite complex at some sections, I would recommend familiarising yourself with the tables `survey_responses`, `report_survey_comments`, and `report_list` as well as the survey tables and variants, `form_list`, `question_list`, `survey_variant_types`, and `survey_question_variants`. There should be descriptions on each relevant field in this plugin guide, as well as the Main survey guide.

### Function get_resident_vs_family_rating_answers:
The queries within this function works very similarly to the get_rating_answers, the main difference is that it makes use a of comparison using a variant of a survey instead of a question.
So in short, responses are split in 2, the main survey typically named "Resident" after the category "Resident", and the variant survey name "Family" after the variant category.
Both labels Resident and Family labels can be changed in the report's options section.

![image](https://github.com/user-attachments/assets/4f238ae3-76f1-424d-a3b8-ffc2075c07aa)


### Function get_comparison_question:
This function simply queries the row that matches the `q_uid` of the comparison question if there is a comparison. Its used in most reporting functions where a comparison question is needed. 

### Function return_filter:
The two locations this function is used in is the processes `process/download-excel-comparison` and `process/download-excel-matrix`, both used to generate excel reports. Its used to query and filter out the responses based on previously selected filter options in the report's options section.

### Function get_potential_filter_options:
This function queries the survey of a report to get all question records from the table `question_list` with the question type `radio`, `checkbox` and `conditional_radio`, the options of these question types can potentially be used as filter options.
There is a real reason for querying these records, you would have to take a look at argument `$radio_questions` the function `get_rating_answers`, which this `get_potential_filter_options` would typically get passed though, and note how the argument gets applied. Note that the variable name `$radio_questions` is legacy, originally only radio question types would be accepted as filter options, but now we accept `radio`, `checkbox` and `conditional_radio` question types.
