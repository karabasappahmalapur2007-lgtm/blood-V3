LIFELINK — BLOOD DONOR & EMERGENCY COORDINATION PORTAL
======================================================

WHAT'S IN THIS BUILD
--------------------
- Separate Donor, Patient/Needer, and Doctor/Hospital login portals.
- Role-protected dashboards and admin-only management.
- Donor profiles, availability controls and consent-based in-app emergency alerts.
- Donor search by blood group and city; lists every available matching donor.
- Public donor profiles hide phone numbers and email addresses.
- Karnataka city dropdown plus distinct cities already present in the SQLite database.
- Emergency request board with 1–20 units and Critical/Urgent/Routine priority.
- Compatible donor matching is a preliminary red-cell filter; same-city opted-in donors get in-app notifications.
- Donors can accept eligible notified requests; patients can track/cancel pending requests; admin can complete accepted requests.
- Admin statistics and downloadable CSV request report.
- CSRF checks, password hashing, role checks, basic request rate limiting, security headers and private authenticated-page caching.
- Render deployment files: render.yaml, Procfile, Gunicorn, persistent-disk SQLite path and /healthz health check.

LOCAL WINDOWS / VS CODE SETUP
-----------------------------
1. Extract the ZIP and open the prgt1 folder in VS Code.
2. In PowerShell:

   py -m venv .venv
   .venv\Scripts\Activate.ps1
   py -m pip install -r requirements.txt

   If PowerShell blocks activation, use the venv interpreter directly:
   .venv\Scripts\python.exe -m pip install -r requirements.txt
   .venv\Scripts\python.exe -m flask --app app create-admin
   .venv\Scripts\python.exe app.py

3. Set a random local secret for the current terminal:

   $env:LIFELINK_SECRET_KEY = (py -c "import secrets; print(secrets.token_hex(32))")

4. Create the first administrator (only once):

   py -m flask --app app create-admin

   Enter the admin name, email and a strong password when prompted.

5. Run locally:

   py app.py

6. Visit http://127.0.0.1:5000

BACK UP / EXISTING DATABASE
---------------------------
- This ZIP intentionally does not contain private donor records.
- Before first launch, back up your old lifelink.sqlite3 file.
- To retain old records, copy that database into this folder as lifelink.sqlite3 before starting the app.
- The app creates missing tables/columns and upgrades the older users role constraint for doctor/hospital accounts.
- For a safer migration, test with a COPY of your database first. Keep the untouched backup offline.

DEPLOY TO RENDER (RECOMMENDED: BLUEPRINT + PERSISTENT DISK)
-----------------------------------------------------------
IMPORTANT: SQLite data on a normal ephemeral Render filesystem can be lost on redeploy/restart. The included render.yaml provisions a persistent disk at /var/data and stores the database at /var/data/lifelink.sqlite3. Persistent disks require a paid Render web-service plan; the Blueprint uses the Starter plan. Do not remove the disk if you need to keep your records.

1. Create a GitHub repository and upload the contents of this prgt1 folder (not the outer ZIP folder).
2. In Render, choose New + > Blueprint and connect that GitHub repository.
3. Render detects render.yaml. Review the service and persistent disk settings, then apply the Blueprint.
4. Wait for the first deploy to finish. Render generates LIFELINK_SECRET_KEY automatically.
5. Open the service URL and check /healthz. It should return {"status":"ok"}.
6. Create the first admin from Render Shell:

   python -m flask --app app create-admin

   If Render Shell is unavailable on your plan, use a one-off protected admin bootstrap process before opening registration to users. Never add a public admin-registration route.
7. Open the website URL, create donor/patient/doctor/hospital accounts, and test the workflows with dummy data first.

MANUAL RENDER SERVICE (IF NOT USING BLUEPRINT)
----------------------------------------------
- Runtime: Python
- Build command: pip install -r requirements.txt
- Start command: gunicorn wsgi:app --workers 2 --threads 4 --timeout 120
- Health check path: /healthz
- Add a persistent disk mounted at /var/data (1 GB or larger).
- Environment variables:
  RENDER=true
  LIFELINK_HTTPS=1
  LIFELINK_DB=/var/data/lifelink.sqlite3
  LIFELINK_SECRET_KEY=<generate a long random secret>
- Keep the database on the mounted disk. Never commit the database or secret to GitHub.

BEFORE REAL-WORLD USE — IMPORTANT LIMITATIONS
---------------------------------------------
This remains an academic coordination prototype, not a certified blood bank, hospital system or emergency dispatch service. Do not use it as the sole source for emergency care. Always contact a hospital or licensed blood bank.

- Blood group matching is only a preliminary red-cell compatibility filter. It does not establish medical suitability; clinical staff must verify donor eligibility, identity, testing and cross-match.
- Notifications are currently in-app only. SMS, WhatsApp, email delivery, OTP, Google Maps geocoding, automated distance calculations and payment integrations are NOT configured in this ZIP and require external providers/API credentials and additional implementation.
- The city filter is a city-level match, not a live GPS radius calculation.
- The current database is SQLite. The included Render persistent disk protects the file across normal deploys, but a multi-instance/high-traffic production system should migrate to managed PostgreSQL and add tested backup/restore procedures.
- Before handling real personal/health-related information, obtain legal/privacy review, add verified institution workflows, monitoring, audit trails, data-retention controls, incident response and security testing.
- Do not publicly expose donor phone numbers or email addresses. The public profile intentionally hides them.

SECURITY NOTES
--------------
- Set a unique LIFELINK_SECRET_KEY in production; production startup fails if it is missing.
- Render TLS proxy headers are handled by ProxyFix; secure cookies are enabled with LIFELINK_HTTPS=1.
- Keep admin credentials private and use a unique strong password.
- The built-in rate limiter is a basic application-level control, not a substitute for a production WAF or distributed rate limiting.
