# infosys-maritime-ai
# Agentic AI for Maritime Freight Pricing and Route Optimization
# Milestone 1
FreightQuote AI is a Streamlit-based authentication module developed for **Infosys Springboard Internship 7.0, Batch 4**. This milestone covers account registration, login, password recovery, session management, and user dashboards.

## Features
- User registration with username, email, password, and a security question.
- Login with username or email.
- Password validation and bcrypt hashing for passwords and security answers.
- Password recovery through a security question or email OTP.
- JWT-based sessions with logout support.
- User and administrator dashboards backed by SQLite.
- Gmail SMTP for OTP delivery.

## Tech Stack
Python, Streamlit, SQLite, bcrypt, PyJWT, Gmail SMTP, Google Colab, and ngrok.

## Run the Project
This project is configured to run in Google Colab.

1. Open the notebook in Google Colab.
2. Add the following values under **Colab Secrets** and enable notebook access for each secret:

   | Secret | Purpose |
   | --- | --- |
   | `JWT_SECRET` | Signs JWT session tokens |
   | `NGROK_AUTHTOKEN` | Authenticates the ngrok tunnel |
   | `EMAIL_ADDRESS` | Gmail address used to send OTPs |
   | `EMAIL_PASSWORD` | Gmail App Password |
   | `ADMIN_EMAIL` | Optional: email assigned administrator access |

3. Enable 2-Step Verification on the sending Gmail account and generate a Gmail App Password.
4. Run the notebook cells in order. Open the public URL printed by ngrok to use the application.

The Colab session must remain running while the application is in use. The ngrok URL may change when a new session starts.

## Password Rules
Passwords must be at least eight characters long and include an uppercase letter, a lowercase letter, a number, and a special character. The confirmation password must match.

## Milestone1 Structure
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

## Security Notes
- Do not commit secrets, OTPs, passwords, or private tokens to GitHub.
- Clear notebook outputs before uploading the notebook.
- The SQLite database is created locally when the application runs; do not upload it if it contains user data.

## Milestone Scope
Milestone 1 focuses on authentication and dashboards. Freight-rate calculation and quotation generation are outside the scope of this milestone. In the current notebook, security-question recovery starts with the registered email address, and administrator access is assigned through `ADMIN_EMAIL`.

