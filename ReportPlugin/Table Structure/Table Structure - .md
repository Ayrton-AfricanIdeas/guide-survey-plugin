# Reports
The table below represents the fields of the table `report_list` and holds the settings for each report. Further explaination on what the fields are needed for are described in detail below.

Table: `report_list`
| Field Name    | Description |
| -------- | ------- |
| user_id  | Not necessarily needed as of yet, but might be needed in future enhancements.   |
| company_id | This field is needed to separate reports by company so that only users of that specific company are able to manage reports made by them.     |
| report_title    | Stores a string representing the title of the report    |
| resident_name | This field is only used if the user needs to change the name of the "Resident" name used in a Resident vs Family comparison report. A Resident survey is an original survey that has the category "resident" |
| family_name | This field is only used if the user needs to change the name of the "Family" name used in a Resident vs Family comparison report. A Family survey is a variant with the category "family". |
| survey_code | The `survey_code` field refers to the unique field `form_code` in the table `form_list`. This field is used to return all questions in the table `question_list` and returns all responses from the table `survey_responses` to later be queried in the report. |
| filters | This field is a json strucuture, please refer below for a detailed explaination on its structure. This json object is used to filter responses used in the report by predetermined options selected by a user who is able to manage the report. |
| comparison| This is is a json object, please refer below for a detailed explaination on its structure. This object is used to define the selected comparison type for the report |
| logo_url | This field is a string field simply used to refer to the logo, stored in the media library, the company would like to use in their report. |

### Json Structure - filter:
The `filter` json object can take multiple elements, each one representing a filter but only 1 option per question.
```
{
    "Unique identifier of the filter question": {
        "key": "Selected option responses should be filtered by",
        "label": "Title of the filter question",
        "title": "Used as a short describer for us to identify the filter question order and question type",
        "pageOrder": "Question order of the filter question"
    },
    "5f538b04-98b6-4d98-8f4e-0be0b2fa781f": {
        "key": "3",
        "label": "Example Filter Question?",
        "title": "Q3: radio",
        "pageOrder": "3"
    }
}
```

### Json Structure - comparison:

```
{
    "comparison": "Unique identifier of comparison question or comparison survey",
    "comparison_type": "Comparison type can be a question type amoung the following: radio, checkbox, conditional_radio. Or variant survey with the category 'family'"
}
```
Example:
```
{
    "comparison": "deadce60-b8e5-4f93-9d45-6dab50d13a1c",
    "comparison_type": "radio"
}
```
