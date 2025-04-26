from flask import Flask, render_template
import pandas as pd
from datetime import datetime, timedelta

app = Flask(__name__)

@app.route('/')
def dashboard():
    # Sample AMC/CMC data
    amc_data = pd.DataFrame({
        'Equipment_ID': ['EQ001', 'EQ002', 'EQ003', 'EQ004', 'EQ005'],
        'Equipment_Name': ['ECG Machine', 'X-Ray Machine', 'Infusion Pump', 'Defibrillator', 'Ultrasound Scanner'],
        'Department': ['Cardiology', 'Radiology', 'ICU', 'Emergency', 'Obstetrics'],
        'AMC_Start_Date': pd.to_datetime(['2023-01-01', '2022-06-15', '2024-01-10', '2023-09-01', '2023-03-20']),
        'AMC_End_Date': pd.to_datetime(['2025-01-01', '2024-06-15', '2025-01-10', '2024-09-01', '2024-03-20'])
    })

    # Sample Breakdown data
    breakdown_data = pd.DataFrame({
        'Equipment_ID': ['EQ001', 'EQ001', 'EQ002', 'EQ002', 'EQ004', 'EQ005', 'EQ005'],
        'Date': pd.to_datetime(['2024-10-12', '2025-02-20', '2023-11-25', '2024-03-10', '2024-12-18', '2023-07-05', '2023-09-22']),
        'Issue_Description': [
            'Display not working', 'Battery failure', 'Tube misalignment',
            'Software freeze', 'Shock delivery failure', 'Image distortion', 'Power issue'
        ],
        'Downtime_in_Days': [2, 1, 3, 2, 4, 2, 1]
    })

    today = datetime.today()

    amc_data['Days_to_AMC_Expiry'] = (amc_data['AMC_End_Date'] - today).dt.days
    amc_data['AMC_Status'] = amc_data['Days_to_AMC_Expiry'].apply(
        lambda x: 'Expired' if x < 0 else ('Expiring Soon' if x <= 30 else 'Valid')
    )

    six_months_ago = today - timedelta(days=180)
    recent_breakdowns = breakdown_data[breakdown_data['Date'] >= six_months_ago]

    breakdown_counts = recent_breakdowns['Equipment_ID'].value_counts()
    frequent_issues = breakdown_counts[breakdown_counts > 2].index.tolist()

    amc_data['Maintenance_Recommended'] = amc_data['Equipment_ID'].apply(
        lambda eq_id: 'Yes' if eq_id in frequent_issues or 
        amc_data[amc_data['Equipment_ID'] == eq_id]['AMC_Status'].values[0] in ['Expired', 'Expiring Soon']
        else 'No'
    )

    dashboard_data = amc_data[['Equipment_ID', 'Equipment_Name', 'Department', 'AMC_Status', 'Maintenance_Recommended']].to_dict(orient='records')

    return render_template('dashboard.html', equipment=dashboard_data)

if __name__ == '__main__':
    app.run(debug=True)
