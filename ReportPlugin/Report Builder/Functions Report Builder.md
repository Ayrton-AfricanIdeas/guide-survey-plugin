# Report Builder Functions
These functions are created on the file `functions/report-builder-comments.php` in the plugin.

### Function report_builder_comment:

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
