# Forgot Password Feature - TaxMate App

## Overview
The forgot password feature allows users to reset their passwords via email verification.

## How it Works

### 1. User Flow
1. User clicks "Forgot Password?" on the login page
2. User enters their email address
3. User receives a password reset email to their inbox (with fallback console output)
4. User clicks the reset link in the email or enters the token manually
5. User sets a new password
6. User can login with the new password

### 2. Backend Endpoints

#### POST /forgot-password
- **Body**: `{"email": "user@example.com"}`
- **Response**: `{"message": "If the email exists, a reset link has been sent"}`
- **Function**: Generates a reset token and sends email (or prints to console in dev)

#### POST /reset-password
- **Body**: `{"token": "reset_token", "new_password": "new_password"}`
- **Response**: `{"message": "Password reset successfully"}`
- **Function**: Validates token and updates user password

### 3. Security Features
- Reset tokens expire after 1 hour
- Tokens are single-use (deleted after successful reset)
- Email existence is not revealed for security
- Passwords are properly hashed using bcrypt

### 4. Development Setup

#### Email Configuration
The system is now configured to send actual emails using Gmail SMTP:

1. **Current Setup**:
   - Email: ahnafk8@gmail.com
   - App password configured
   - SMTP settings ready for Gmail

2. **Email Behavior**:
   - Attempts to send real email first
   - Falls back to console output if email fails
   - Reset tokens printed to console for development backup

1. **Gmail Setup**:
   - Enable 2FA on Gmail account
   - Generate app password
   - Update SMTP settings in `main.py`

2. **Alternative Services**:
   - SendGrid API
   - AWS SES
   - Mailgun
   - Outlook SMTP

#### Testing
1. Start the backend server
2. Use the forgot password feature in the app
3. Check your email inbox AND spam/junk folder
4. Check console output for the reset token and detailed SMTP logs
5. Use the token to reset password

#### Troubleshooting Email Issues
If emails aren't arriving:

1. **Check Spam/Junk Folder**: Gmail often puts automated emails there
2. **Gmail Security**: Gmail may block emails sent from the same domain
3. **App Password**: Try generating a new app password
4. **Console Logs**: Check server output for detailed SMTP debug info
5. **Alternative**: Use the printed token from console for testing

**Quick Fixes:**
- Generate new Gmail app password
- Check spam folder
- Try sending to a different email provider (Yahoo, Outlook)
- Use console token as backup

### 5. Files Added/Modified

#### Frontend (Flutter)
- `forgot_password_page.dart` - Forgot password form
- `reset_password_page.dart` - Reset password form
- `login_page.dart` - Added forgot password link
- `api_service.dart` - Added API methods
- `main.dart` - Added route handling

#### Backend (FastAPI)
- `main.py` - Added forgot/reset password endpoints
- `email_config.py` - Email configuration reference

### 6. Production Considerations
- Store reset tokens in database or Redis instead of memory
- Use a proper email service (SendGrid, AWS SES)
- Add rate limiting for forgot password requests
- Log security events
- Add CAPTCHA for additional security
- Use HTTPS for all password-related operations

### 7. Usage Instructions

#### For Users
1. On login page, click "Forgot Password?"
2. Enter your email address
3. Check your email for reset link (or check console in development)
4. Click link or manually navigate to reset page
5. Enter new password and confirm
6. Login with new password

#### For Developers
1. Configure email settings in `main.py`
2. Test with various email scenarios
3. Monitor console output in development
4. Verify token expiration works correctly
5. Test password reset flow end-to-end
