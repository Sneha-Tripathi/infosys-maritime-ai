# infosys-maritime-ai
# Agentic AI for Maritime Freight Pricing and Route Optimization
# Milestone 1

FreightQuote AI is a Streamlit-based authentication module developed for **Infosys Springboard Internship 7.0, Batch 4**. Milestone 1 covers account registration, login, password recovery, session management, and user and administrator dashboards.

## Features

* User registration and login using a username or email address
* Password validation and bcrypt hashing
* Password recovery using a security question or email OTP
* JWT-based session management and logout
* SQLite database and role-based dashboards
* OTP delivery through Gmail SMTP

## Tech Stack

Python, Streamlit, SQLite, bcrypt, PyJWT, Gmail SMTP, Google Colab, and ngrok.

## Configuration

Run the project in Google Colab. Open the **Secrets** panel using the key icon in the left sidebar. Add the following secrets and enable notebook access for each one.

| Secret            | Purpose                                                                |
| ----------------- | ---------------------------------------------------------------------- |
| `JWT_SECRET`      | Signs JWT session tokens                                               |
| `NGROK_AUTHTOKEN` | Authenticates the ngrok tunnel                                         |
| `EMAIL_ADDRESS`   | Gmail address used to send OTPs                                        |
| `EMAIL_PASSWORD`  | Gmail App Password                                                     |
| `ADMIN_EMAIL`     | Email of a registered account that should receive administrator access |

### 1. JWT Configuration

The application uses PyJWT to sign and verify authentication tokens. Tokens use the HS256 algorithm and expire after two hours.

Generate a strong signing secret in a temporary Google Colab cell:

```python
import secrets
print(secrets.token_urlsafe(32))
```

Copy the generated value into Colab Secrets under `JWT_SECRET`. Keep the secret private and do not commit it to GitHub. Use the same secret across sessions unless you intend to invalidate existing tokens.

### 2. ngrok Configuration

1. Create an account at [ngrok](https://ngrok.com/).
2. Copy your authtoken from the [ngrok dashboard](https://dashboard.ngrok.com/get-started/your-authtoken).
3. Save it in Colab Secrets as `NGROK_AUTHTOKEN`.
4. Enable notebook access for the secret.

The notebook uses this token to expose the Streamlit application through a public URL.

### 3. Gmail SMTP Configuration

1. Sign in to the Gmail account used to send OTP emails.
2. Enable **2-Step Verification** in Google Account settings.
3. Generate a Gmail App Password.
4. Save the Gmail address as `EMAIL_ADDRESS` and the App Password as `EMAIL_PASSWORD` in Colab Secrets.

The application sends emails through `smtp.gmail.com` on port `587` using STARTTLS. Do not use your regular Gmail password or commit email credentials to GitHub.

### 4. Administrator Access

Set `ADMIN_EMAIL` to the exact email address of an account already registered in the application. Enable notebook access and restart the application to load the setting.

## Run the Application

1. Open `Milestone1.ipynb` in Google Colab.
2. Configure the required Colab Secrets.
3. Run the notebook cells in order: install dependencies, write `app.py`, validate the code if applicable, and start the application.
4. Open the public URL printed by ngrok.

Keep the launcher cell running while using the application. The public URL may change when a new session starts.

## Password Rules

Passwords must contain at least eight characters, including an uppercase letter, a lowercase letter, a digit, and a special character. The confirmation password must match.

## Project Structure

```text
Milestone1/
├── README.md
├── Milestone1.ipynb
└── screenshots/
    ├── login.png
    ├── signup.png
    ├── forgot-password-security-question.png
    ├── forgot-password-email-otp.png
    ├── otp-email.png
    ├── user-dashboard.png
    └── admin-dashboard.png
```

Add the actual screenshots to the `screenshots/` folder and ensure their filenames match the references above.

## Security Notes

* Never commit passwords, OTPs, JWT signing keys, ngrok tokens, or other secrets to GitHub.
* Clear notebook outputs that may contain sensitive information before committing.
* Do not upload a SQLite database containing user information.
* The application runs in a temporary Colab environment and is intended for milestone demonstration.

## Milestone Scope

Milestone 1 focuses on authentication, password recovery, session management, and dashboards. Freight-rate calculation and quotation generation are outside the scope of this milestone.

