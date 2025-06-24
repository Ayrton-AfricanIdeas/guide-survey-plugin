### Generate Excel Comment Comparison
This process can be found `process/download-excel-comparison.php`.
The contents of this process makes heavy use of functions from `functions/functions_query_doc_gen.php`, it would be beneficial to first review it [here](https://github.com/Ayrton-AfricanIdeas/guide-survey-plugin/blob/main/ReportPlugin/Functions/Functions%20Document%20Generation.md) before continuing.

### Initialise PhpSpreadsheet
Before carrying out the process to generate the excel report we need include the necessary library and functions, namely `PhpOffice PhpSpreadsheet` and `functions_query_doc_gen.php` that was previously mentioned at the start.
```
include __DIR__ . '/../functions/functions_query_doc_gen.php';
require_once(__DIR__ . '/../vendor/autoload.php');
require_once(dirname(__DIR__, 3) . '/../wp-load.php');

use PhpOffice\PhpSpreadsheet\Spreadsheet;
use PhpOffice\PhpSpreadsheet\Writer\Xlsx;
use PhpOffice\PhpSpreadsheet\Style\Color;
use PhpOffice\PhpSpreadsheet\Style\Border;
use PhpOffice\PhpSpreadsheet\Style\Fill;
use PhpOffice\PhpSpreadsheet\Style\Alignment;

$report_details = get_report_details_for_doc($_GET['report_id']);
$filter = return_filter($report_details);
```
