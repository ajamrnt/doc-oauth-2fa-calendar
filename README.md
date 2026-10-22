# eDoc-COS80029

eDoc doctor appointment system with Google / Microsoft sign-in and calendar sync.

## Calendar sync (Amar)

When a patient books, reschedules or cancels an appointment, the event is added, moved or
removed in their calendar automatically.

- Signed in with Google: the event goes into their Google Calendar
- Signed in with Microsoft: the event goes into their Outlook calendar
- Password users: can connect Google / Outlook from My Appointments, download an `.ics` file,
  or subscribe to their personal calendar feed (Apple Calendar etc.)
- Doctors who signed in with Google or Microsoft also get the bookings in their calendar
- Admin / doctor deletes (appointments, sessions, accounts) remove the events too
- If Google or Microsoft can't be reached, the booking is still saved and the error is logged in `logs/app.log`

Main files:

| File | |
|---|---|
| `lib/calendar.php` | sync logic (Google Calendar API, Microsoft Graph, .ics) |
| `calendar-connect.php` | connect / disconnect a calendar |
| `calendar-feed.php` | subscription feed |
| `patient/calendar-ics.php` | .ics download |
| `patient/booking-complete.php`, `patient/reschedule.php`, `patient/delete-appointment.php` | book / reschedule / cancel + sync |
| `sql/oauth-calendar-migration.sql` | extra tables (`calendar_connections`, `appointment_calendar_events`) |

## Doctor two-factor authentication (Manoj)

Doctors have to use an authenticator app (Google Authenticator, Microsoft Authenticator, Authy).

- First login: the doctor scans a QR code and confirms a 6-digit code, then gets 8 one-time backup codes
- Every login after that (password, Google or Microsoft): `verify-2fa.php` asks for the 6-digit code or a backup code
- Until 2FA is set up, every doctor page sends them back to `doctor/enable-2fa.php`
- Admin: `admin/manage-2fa.php` shows who has 2FA and can reset it, `admin/add-doctor.php` creates a doctor with a temporary password
- Needs the composer packages (`composer install`) and the PHP `gd` extension

## Running it locally (MAMP)

1. Put the project in `/Applications/MAMP/htdocs/edoc` and start MAMP.
2. Run `composer install` in the folder, then open `http://localhost:8888/edoc/setup.php`. It creates `config.php`, the database and the extra tables.
3. Create your own Google and Microsoft keys and put them in `config.php` (see **Keys and secrets** below).
4. Log in at `http://localhost:8888/edoc/login.php` (demo accounts: `patient@edoc.com`, `doctor@edoc.com`, `admin@edoc.com`, password `123`).

## Keys and secrets (bring your own)

This repo has **no** API keys, client secrets or encryption keys in it. Everyone who runs it uses their own:

| Key | Where it comes from | Where it goes |
|---|---|---|
| Encryption key (protects stored calendar tokens) | Made by `setup.php`, different for every install | `config.php` |
| Google client ID + secret | Your own OAuth client in Google Cloud | `config.php` |
| Microsoft client ID + secret | Your own app registration in Azure | `config.php` |
| Database login | Your local MAMP / XAMPP | `config.php` |

`config.php` is created from `config.example.php` (placeholders only) and is in `.gitignore`, so keys never get committed.
Until you add the Google / Microsoft keys, those buttons show a "not available" message and email + password login still works.

**Google**
1. [console.cloud.google.com](https://console.cloud.google.com) → create a project.
2. OAuth consent screen: External, and add your Google account as a test user.
3. APIs & Services → Library → enable **Google Calendar API**.
4. Credentials → Create credentials → OAuth client ID → **Web application**.
   Authorised redirect URI: `http://localhost:8888/edoc/oauth-callback.php`
5. Copy the client ID and secret into `google_client_id` / `google_client_secret`.

**Microsoft**
1. [portal.azure.com](https://portal.azure.com) → App registrations → New registration →
   "Accounts in any organizational directory and personal Microsoft accounts".
2. Redirect URI: platform **Web**, `http://localhost:8888/edoc/microsoft-callback.php`
3. Certificates & secrets → New client secret → copy the **Value** (not the Secret ID).
4. API permissions → Microsoft Graph → Delegated: `openid`, `profile`, `email`, `offline_access`, `User.Read`, `Calendars.ReadWrite`.
5. Copy the Application (client) ID and the secret value into `microsoft_client_id` / `microsoft_client_secret`.

Open `setup.php` again afterwards, it shows a tick when each provider is set up.
If a key ever gets shared by mistake, delete it in Google Cloud / Azure and make a new one. Removing it from the code is not enough.
