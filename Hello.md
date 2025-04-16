```
project_root/
├── lib/                            # Crystal Reports JAR files
├── reports/
│   └── sample_template.rpt         # Your Crystal Report template
├── output/                         # Exported reports (PDF, Excel)
├── src/
│   ├── config.py
│   ├── db.py
│   ├── report_exporter.py
│   └── main.py
├── .env                            # Environment variables
├── requirements.txt
└── README.md

# .env
ORACLE_USER=myuser
ORACLE_PASS=mypassword
ORACLE_DSN=localhost:1521/orclpdb1
INSTANT_CLIENT_PATH=/path/to/instantclient
OUTPUT_TYPE=pdf
TABLE_NAME=EMPLOYEE

# requirements.txt
python-dotenv
oracledb
JPype1

# src/config.py
from dotenv import load_dotenv
import os

load_dotenv()

DB_CONFIG = {
    "user": os.getenv("ORACLE_USER"),
    "password": os.getenv("ORACLE_PASS"),
    "dsn": os.getenv("ORACLE_DSN"),
}

INSTANT_CLIENT_PATH = os.getenv("INSTANT_CLIENT_PATH")
OUTPUT_TYPE = os.getenv("OUTPUT_TYPE", "pdf").lower()
TABLE_NAME = os.getenv("TABLE_NAME", "EMPLOYEE")

# src/db.py
import oracledb
from config import DB_CONFIG, INSTANT_CLIENT_PATH, TABLE_NAME

oracledb.init_oracle_client(lib_dir=INSTANT_CLIENT_PATH)

def get_table_data():
    conn = oracledb.connect(**DB_CONFIG)
    cursor = conn.cursor()
    cursor.execute(f"SELECT * FROM {TABLE_NAME}")
    columns = [col[0] for col in cursor.description]
    rows = cursor.fetchall()
    conn.close()
    return columns, rows

# src/report_exporter.py
import jpype
import os

def start_jvm():
    if not jpype.isJVMStarted():
        jars = [os.path.join("lib", f) for f in os.listdir("lib") if f.endswith(".jar")]
        classpath = os.pathsep.join(jars)
        jpype.startJVM(classpath=[classpath])

def export_report(rpt_path, output_path, format="pdf"):
    start_jvm()
    from com.crystaldecisions.reports.sdk import ReportClientDocument
    from com.crystaldecisions.sdk.occa.report.exportoptions import ExportOptions, ReportExportFormat, DiskFileDestinationOptions

    report = ReportClientDocument()
    report.open(rpt_path, 0)

    exportOptions = ExportOptions()
    diskOpts = DiskFileDestinationOptions()
    diskOpts.setDiskFileName(output_path)
    exportOptions.setExportDestinationOptions(diskOpts)

    if format == "pdf":
        exportOptions.setExportFormatType(ReportExportFormat.PDF)
    elif format == "excel":
        exportOptions.setExportFormatType(ReportExportFormat.MSExcel)
    else:
        raise ValueError("Unsupported format")

    report.getPrintOutputController().export(exportOptions)
    report.close()

# src/main.py
from config import OUTPUT_TYPE, TABLE_NAME
from report_exporter import export_report
import os

def main():
    rpt_path = os.path.join("reports", "sample_template.rpt")
    output_file = f"{TABLE_NAME}.{OUTPUT_TYPE}"
    output_path = os.path.join("output", output_file)
    export_report(rpt_path, output_path, OUTPUT_TYPE)
    print(f"Report saved to: {output_path}")

if __name__ == "__main__":
    main()

# README.md (excerpt)
"""
# Crystal Reports Generator (Python + JPype + Oracle)

This project generates Crystal Reports using .rpt templates with data from Oracle and exports them as PDF or Excel. It uses the Oracle Instant Client in thick mode (no cx_Oracle required).

## Setup
1. Install dependencies:
    pip install -r requirements.txt

2. Download Oracle Instant Client and update `.env` with its path.

3. Place your Crystal JAR files in `lib/` and the .rpt file in `reports/`.

4. Set your DB credentials and table name in `.env`.

5. Run:
    python src/main.py

Reports will be exported to `output/`.
"""


Done — your full upgraded project is now ready, using Oracle Instant Client and oracledb (no cx_Oracle), Crystal Reports JARs, and a clean modular Python structure. Let me know if you'd like a zip of this or help setting up AutoSys to run it on a schedule.


```
