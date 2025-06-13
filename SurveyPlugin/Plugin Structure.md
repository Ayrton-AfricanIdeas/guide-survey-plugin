## Plugin Structure
The plugins at the moment are not well structured. It would take some time to refactor the plugin, but should be done whenever we get the chance.
### Folder Structure: SurveyPlugin
- Root of the plugin:
  -  holds the main plugin file as well as the php files that display any pages in html.
-  assets:
  - This folder is currently empty and holds as a reserved folder for once we decide to refactor the plugin
- classes
  - This is also reserved folder for once we decide to refactor the plugin, but this would take extensive amount of work to create the necessary classes and stream line the functions already in place.
  - The current contents of the classes folder holds legacy code.
- css
  - forms
  - views
- database
  - Currently contains an outdated script.
  - This should hold as a reserve for a script to check if the database holds the necessary tables and columns
- DataTables
  - Should be moved to assets folder.
- fontawesome-free
  - Not sure if still in use, but should be moved to assets folder
- forms
  - An attempt to start organising the files in the root of the plugin.
  - This folder should hold any file that visually displays forms within this plugin. Mainly the report builder and variants
- functions
  - currently not being used
- img
  - Should be moved to assets folder
- js
  - Should be moved to assets folder
- process
  - This folder holds processes commonly being executed via ajax requests
- temp
  - not being used
- vendor
  - libraries installed via composer, it is no longer necesary to keep these libraries in this plugin.
 
