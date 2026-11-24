<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Penetration Testing Report: Mediroza General Hospital</title>
  <style>
    :root {
      --bg: #0f172a;
      --surface: #1e293b;
      --surface-alt: #162032;
      --text: #e2e8f0;
      --muted: #94a3b8;
      --accent: #38bdf8;
      --border: #334155;
      --critical: #ef4444;
      --critical-bg: rgba(239, 68, 68, 0.15);
      --high: #f97316;
      --high-bg: rgba(249, 115, 22, 0.15);
      --code-bg: #0b1120;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      padding: 32px 16px;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
      background-color: var(--bg);
      color: var(--text);
      line-height: 1.65;
    }

    .report-container {
      max-width: 920px;
      margin: 0 auto;
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 36px 40px;
      box-shadow: 0 12px 32px rgba(0, 0, 0, 0.35);
    }

    h1 {
      margin-top: 0;
      font-size: 2rem;
      color: #f8fafc;
      border-bottom: 2px solid var(--accent);
      padding-bottom: 12px;
    }

    h2 {
      margin-top: 32px;
      font-size: 1.35rem;
      color: var(--accent);
      border-bottom: 1px solid var(--border);
      padding-bottom: 6px;
    }

    h3 {
      margin-top: 22px;
      font-size: 1.1rem;
      color: #f1f5f9;
    }

    p {
      margin: 10px 0;
    }

    ul {
      margin: 10px 0 16px;
      padding-left: 22px;
    }

    li {
      margin-bottom: 8px;
    }

    code {
      font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
      background: var(--code-bg);
      color: #7dd3fc;
      padding: 2px 6px;
      border-radius: 4px;
      font-size: 0.9em;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin: 16px 0 24px;
      background: var(--surface-alt);
      border-radius: 8px;
      overflow: hidden;
    }

    th, td {
      padding: 12px 14px;
      text-align: left;
      border-bottom: 1px solid var(--border);
      vertical-align: top;
    }

    th {
      background: #0f172a;
      color: #f8fafc;
      font-weight: 600;
    }

    tr:last-child td {
      border-bottom: none;
    }

    .badge {
      display: inline-block;
      padding: 3px 10px;
      border-radius: 999px;
      font-size: 0.82rem;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: 0.04em;
    }

    .badge-critical {
      color: #fca5a5;
      background: var(--critical-bg);
      border: 1px solid rgba(239, 68, 68, 0.4);
    }

    .badge-high {
      color: #fdba74;
      background: var(--high-bg);
      border: 1px solid rgba(249, 115, 22, 0.4);
    }

    @media (max-width: 640px) {
      .report-container {
        padding: 22px 18px;
      }

      h1 {
        font-size: 1.55rem;
      }
    }
  </style>
</head>
<body>
  <main class="report-container">
    <h1>Penetration Testing Report: Mediroza General Hospital</h1>

    <section id="executive-summary">
      <h2>01 Executive Summary</h2>
      <p>
        A structured manual security assessment was conducted against <code>https://medirozahospital.com</code> and
        <code>www.medirozahospital.com</code> (<code>199.188.201.16:443</code>) running
        <strong>Mediroza CMS 1.4.2</strong> on <strong>PHP/8.2.36</strong> and <strong>LiteSpeed</strong> with a
        <strong>MySQL/MariaDB</strong> backend. The assessment identified critical authentication, file-protection,
        and data-exposure weaknesses across Milestones 1 through 3:
      </p>
      <ul>
        <li>
          <strong>Authentication Bypass via SQL Injection:</strong> Unsanitized input in the <code>username</code>
          parameter of <code>/patient/login.php</code> allowed SQL syntax manipulation (<code>admin' #</code>) to bypass
          login verification and access the patient portal (<code>My Reports</code>).
        </li>
        <li>
          <strong>Weakly Protected Patient Lab Reports:</strong> Three encrypted PDF pathology reports
          (<code>LR-2024-1187</code>, <code>LR-2024-1192</code>, and <code>LR-2024-1205</code>) were accessible from
          the portal and had their passwords recovered using the Networkwalks password hash cracker.
        </li>
        <li>
          <strong>Exposed Database Backup (<code>mediroza_db_backup_2019.sql</code>):</strong> An unauthenticated SQL
          backup dump (<code>mediroza_hr</code>) exposed confidential records for 30 hospital staff members (including
          <code>monthly_salary_zar</code>, <code>national_id</code>, email, and phone details) and 10 shareholders
          (including ownership percentages, share counts, and share classes).
        </li>
      </ul>
    </section>

    <section id="scope-and-methodology">
      <h2>02 Scope and Methodology</h2>
      <ul>
        <li>
          <strong>Target Scope:</strong> <code>https://medirozahospital.com</code> and
          <code>www.medirozahospital.com</code> (<code>199.188.201.16:443</code>).
        </li>
        <li>
          <strong>Identified Stack:</strong> Mediroza CMS 1.4.2, PHP/8.2.36, LiteSpeed web server, MySQL/MariaDB.
        </li>
        <li>
          <strong>Tools &amp; Techniques:</strong> Browser Developer Tools (Network, Request, Cookies, Response
          inspection), manual <code>curl</code> requests, SQL injection comment testing, Networkwalks password hash
          cracker for PDF password recovery, and manual inspection of exposed files and SQL backup contents.
        </li>
        <li>
          <strong>Testing Approach &amp; Limitations:</strong> Testing was performed manually with controlled single
          requests within the authorized target scope without automated flooding or denial-of-service activity.
        </li>
      </ul>
    </section>

    <section id="findings-and-exploitation">
      <h2>03 Findings and Proof of Exploitation</h2>

      <h3>Finding 1: SQL Injection and Authentication Bypass in <code>/patient/login.php</code> (Milestone 1)</h3>
      <p>
        <strong>Description:</strong> Submitting a single quote (<code>'</code>) in <code>username</code> on
        <code>/patient/login.php</code> returned HTTP <code>200</code> with a MySQL syntax error inside
        <code>&lt;div class="alert"&gt;</code>:
        <code>Warning: mysqli_query(): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '&amp;#039;' at line 1</code>.
        Testing <code>admin</code> with an empty password returned <code>Incorrect password</code>, confirming the
        account existed.
      </p>
      <p>
        <strong>Proof of Exploitation:</strong> Sending <code>username=admin' #</code> in a <code>POST</code> request to
        <code>/patient/login.php</code> bypassed authentication and rendered
        <code>My Reports | Mediroza Patient Portal</code> under an active session
        (<code>PHPSESSID=sp1ahrsm1sj3uaisnpt345t75s</code>), exposing three encrypted lab reports:
      </p>
      <ul>
        <li><code>Pathology Report - S. Dlamini</code> (<code>Lab Ref LR-2024-1187</code>, <code>2024-11-04</code>, <code>download.php?id=1</code>)</li>
        <li><code>Pathology Report - P. Reddy</code> (<code>Lab Ref LR-2024-1192</code>, <code>2024-11-05</code>, <code>download.php?id=2</code>)</li>
        <li><code>Pathology Report - E. Thompson</code> (<code>Lab Ref LR-2024-1205</code>, <code>2024-11-06</code>, <code>download.php?id=3</code>)</li>
      </ul>

      <h3>Finding 2: Recovery of Passwords for Encrypted Patient PDF Reports (Milestone 2)</h3>
      <p>
        <strong>Description:</strong> The three pathology PDFs downloaded from <code>download.php?id=1</code>,
        <code>download.php?id=2</code>, and <code>download.php?id=3</code> were encrypted.
      </p>
      <p>
        <strong>Proof of Exploitation:</strong> Using the Networkwalks password hash cracker, the passwords for all three
        encrypted PDF lab reports were recovered, allowing access to the report contents for inspection.
      </p>

      <h3>Finding 3: Unauthenticated Exposure of Confidential HR and Shareholder Database Dump (Milestone 3)</h3>
      <p>
        <strong>Description:</strong> The plain-text backup file <code>mediroza_db_backup_2019.sql</code> (generated by
        Mediroza CMS 1.4.2 for database <code>mediroza_hr</code>, dated <code>2019-08-27 02:14:03</code>) was accessible,
        containing full table definitions and <code>INSERT</code> statements for <code>staff</code> and
        <code>shareholders</code>.
      </p>
      <p>
        <strong>Proof of Exploitation:</strong> Inspection of <code>mediroza_db_backup_2019.sql</code> revealed:
      </p>
      <ul>
        <li>
          <strong><code>staff</code> Table (30 records):</strong> Includes <code>id</code>, <code>full_name</code>,
          <code>job_title</code>, <code>department</code>, <code>email</code>, <code>phone</code>,
          <code>national_id</code>, <code>monthly_salary_zar</code>, and <code>date_joined</code> (for example,
          <code>Dr. Johan van der Merwe</code> at <code>160000</code> ZAR/month, <code>Sarah Botha</code> at
          <code>152000</code> ZAR/month, <code>Dr. Rajesh Naidoo</code> at <code>138000</code> ZAR/month, down to
          <code>Linda Fourie</code> at <code>19000</code> ZAR/month).
        </li>
        <li>
          <strong><code>shareholders</code> Table (10 records):</strong> Includes <code>id</code>,
          <code>shareholder_name</code>, <code>share_percent</code>, <code>shares_held</code>, and
          <code>share_class</code> (<code>Ordinary</code> and <code>Preferential</code>), led by
          <code>Dr. Rajesh Naidoo</code> (<code>18.0%</code>, <code>180000</code> shares),
          <code>Cedar Health Holdings (Pty) Ltd</code> (<code>15.0%</code>, <code>150000</code> shares),
          <code>Dr. Johan van der Merwe</code> (<code>12.0%</code>, <code>120000</code> shares), and
          <code>Reddy Family Trust</code> (<code>11.0%</code>, <code>110000</code> shares).
        </li>
      </ul>
    </section>

    <section id="risk-rating">
      <h2>04 Risk Rating</h2>
      <table>
        <thead>
          <tr>
            <th>Finding</th>
            <th>Severity</th>
            <th>Justification</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td><strong>Exposed <code>mediroza_db_backup_2019.sql</code> (<code>staff</code> salaries, IDs, and <code>shareholders</code>)</strong></td>
            <td><span class="badge badge-critical">Critical</span></td>
            <td>Directly exposes sensitive employee compensation, national IDs, contact details, and corporate ownership data in plain text without requiring authentication.</td>
          </tr>
          <tr>
            <td><strong>SQL Injection Authentication Bypass in <code>/patient/login.php</code></strong></td>
            <td><span class="badge badge-critical">Critical</span></td>
            <td>Allows unauthenticated attackers to bypass login controls via SQL comment injection (<code>admin' #</code>) and gain access to restricted patient portal resources.</td>
          </tr>
          <tr>
            <td><strong>Weakly Protected Encrypted Patient Lab Reports</strong></td>
            <td><span class="badge badge-high">High</span></td>
            <td>Encrypted lab report passwords were cracked using the Networkwalks password hash cracker, exposing sensitive medical/pathology records once retrieved.</td>
          </tr>
        </tbody>
      </table>
    </section>

    <section id="recommendations">
      <h2>05 Recommendations and Remediation</h2>
      <ul>
        <li>
          <strong>Use Parameterized Queries / Prepared Statements:</strong> Replace direct string concatenation in
          <code>mysqli_query()</code> on <code>/patient/login.php</code> (and across the application) with parameterized
          prepared statements (<code>mysqli_prepare</code> / <code>bind_param</code> or PDO), and suppress verbose
          database error output in production (<code>display_errors = Off</code>).
        </li>
        <li>
          <strong>Remove Publicly Accessible Backups and Enforce Access Controls:</strong> Immediately remove
          <code>mediroza_db_backup_2019.sql</code> and any other backup archives from web-accessible directories, store
          backups outside the document root or in restricted storage, and require strict authentication and
          authorization checks on all endpoints.
        </li>
        <li>
          <strong>Strengthen Document Encryption and Access Controls:</strong> Use strong encryption and strong, unique
          passwords for sensitive PDF documents, avoid relying solely on password-protected files as a substitute for
          proper authorization controls on <code>download.php</code>, and rotate any exposed credentials or sensitive
          identifiers as appropriate.
        </li>
      </ul>
    </section>
  </main>
</body>
</html>
