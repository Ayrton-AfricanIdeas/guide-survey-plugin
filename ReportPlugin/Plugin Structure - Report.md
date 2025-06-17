
# Folder Structure: survey-filter-plugin
The following folder structure is much more organised than the SurveyPlugin folder since it was made after. For now this plugin structure should server as the standard until the plugins are refactored.
- Root of the plugin:
  - The structural order of files within this plugin should be much more organised than the main plugin due to the fact that the purpose and functions needed for this plugin was clear enough to structure the folders accordingly as I've already written similar code in the main plugin.
- assets
  - bootstrap
  - fontawesome-free
  - logos
  - marks
  - stars
  - style
- css
  - Should be moved to style folder when given a chance
- functions
  - This folder holds functions used in conjuction with certain template files and processes
- js
  - should be moved to assets folder
- process
  - This folder holds processes commonly being executed via ajax requests
- template
  - This is an umbrella folder holding any file that displays a page.
- vendor
  - Stores libraries installed via Composer. phpSpreadsheets is the main library needed to generate the excel reports
