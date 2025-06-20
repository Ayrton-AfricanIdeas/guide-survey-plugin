# Report Builder Functions
These functions are created on the file `functions/report-builder-comments.php` in the plugin.

Before getting into the functions, it might be a good idea to explain how comparison questions work.
As you can see below, we have a radio option you can select, when we edit the report we can set this question as a comparison question.
![image](https://github.com/user-attachments/assets/28d592d9-e380-4714-8d59-2c6b1ec2959f)
Then the when answering the star-rating question we leave a comment:
![image](https://github.com/user-attachments/assets/c800aa85-c5a6-47b0-8e4d-dc90ea4f48cb)

In the report builder, we should see the report analysis and the label referencing the radio option we selected before answering that star-rating question.
![image](https://github.com/user-attachments/assets/1cf073f0-b617-4e65-bafb-883472495877)

### Function report_builder_comment:
This function is used to allow users to include and edit comments in a pdf report for a specific star-rating question, as well as being able to change the sentiment of a comment, this only affects the Comment Excel Document generator (see `process/download-excel-comparison.php`)

The function `report_builder_comment` is used in the file `templates/report-view.php` but the file is included in the file `templates/report-builder.php`.
To refer to the argument `$report_array` that is being used when the function is called in `templates/report-view.php`, you can find the variable `$responses` in the file `templates/report-builder.php` that is being passed as that argument. 

We need to ensure that when we retrieve the star-rating responses from the array that we only retrieve answers where users left a comment.
A typical answer field for a star-rating question might look like this if a user left a comment: `{"3":"They did a great job!"}`.
And a answer field for a star-rating question might look like this if a user decided to **not** leave a comment: `{"2":""}`.
So all answers without comments will be excluded from the array.
```
    $filtered_report_array = array_filter($report_array, function ($record) {
        $decoded_answer = json_decode($record['answer'], true);
        // Make sure it's actually an array before using reset()
        if (is_array($decoded_answer)) {
            return reset($decoded_answer) !== '';
        }
    });
```
The rest of the function generates the comment records for a user to edit or view.
![image](https://github.com/user-attachments/assets/87654875-9f7d-4032-b1f4-fb1b5c745cca)


### Function report_analysis:
The report analysis function generates a string used only in the PDF report that summarises the results of a specific star-rating question. The results of the string is dynamic and makes use of `comparison_value`, if the report is a comparison report, to seperate results that match each other on that value.

The `report_analysis` function makes use of a helper function from a different file (see `functions/functions_query_doc_gen.php`) called `get_comparison_question`:
```
$comparison = get_comparison_question($form_id, $comparison_id);
```
The variable `$comparison` that gets initialised is useful to identify which option a user selected from the comparison question so that we can label the comments with the comparison option a user might have selected when answering a survey.

