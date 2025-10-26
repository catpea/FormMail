# 📜 FormMail – Internet History in Perl

**Version:** 1.92
**Author:** Matt Wright
**Original Release:** 1995
**Archive:** [Matt’s Script Archive](http://www.scriptarchive.com/formmail.html)

---

## 🧠 What Was FormMail?

Before PHP, Node.js, or serverless functions, **FormMail** was *the* way websites collected user feedback and sent it by email.

If you visited a personal homepage in the 1990s or early 2000s and filled out a *Contact Us*, *Guestbook*, or *Feedback* form — odds are it was powered by **FormMail.pl** running quietly on a Unix web server.

FormMail was written in **Perl**, one of the earliest and most widely installed web scripting languages, and executed as a **CGI (Common Gateway Interface)** script. When a user submitted a form, the web server passed the input fields to FormMail through environment variables or standard input — and the script handled the rest: parsing, validation, and sending an email with the form contents.

FormMail helped democratize early web interactivity — no databases, no frameworks, just a form and a single script.

---

## ⚙️ How FormMail Worked

At its core, FormMail is a *flow-controlled pipeline* for handling a single form submission.
The execution order was straightforward but carefully orchestrated:

```perl
&check_url;
&get_date;
&parse_form;
&check_required;
&send_mail;
&return_html;
```

Let’s break that down.

---

### 1. 🔒 `check_url`

**Purpose:** Ensure the form submission came from an authorized website.

FormMail checked the `HTTP_REFERER` environment variable — the URL of the page that submitted the form — and compared it against an allowed list in the `@referers` array:

```perl
@referers = ('example.com', '123.45.67.89');
```

This was meant to prevent *open relay abuse*, where spammers would use an unprotected FormMail installation to send bulk email through someone else’s server.

---

### 2. 🕒 `get_date`

Captured and formatted the current date and time for email headers and receipts:

```perl
$date = "Monday, January 10, 2000 at 14:23:05";
```

It used Perl’s built-in `localtime()` to construct human-readable timestamps.

---

### 3. 🧩 `parse_form`

This was the engine room of FormMail.

It read form inputs either from:

* The query string (for `GET` requests), or
* The request body (for `POST` requests).

Each field was split by `&`, then by `=`, and decoded from URL encoding (`%20` → space).

FormMail sorted form data into two associative arrays:

* `%Config` – Reserved fields for configuration (like `recipient`, `subject`, `redirect`).
* `%Form` – User-provided fields (like `email`, `message`, `name`).

It also sanitized and cleaned up the inputs to prevent null bytes and simple injection attacks.

---

### 4. ✅ `check_required`

Ensured mandatory fields were present.

Forms could specify which fields were required:

```html
<input type="hidden" name="required" value="email,realname,message">
```

If any of those were empty, FormMail generated an error page (or redirected to a specified error URL).

It also validated email syntax via the simple regex of its time:

```perl
$email !~ /^.+\@[\w\.\-]+\.[A-Za-z]{2,3}$/
```

In today’s standards, this was a gentle check — but for the 1990s, it was effective enough.

---

### 5. ✉️ `send_mail`

This step piped the message directly into the Unix `sendmail` program:

```perl
open(MAIL, "|$mailprog");
print MAIL "To: $Config{'recipient'}\n";
print MAIL "From: $Config{'email'} ($Config{'realname'})\n";
print MAIL "Subject: $Config{'subject'}\n\n";
```

Then it printed the submitted form fields as plain text, optionally including selected environment variables (IP address, browser, hostname, etc.).

The `sendmail` process would then queue and deliver the message to the intended recipient.

---

### 6. 🌐 `return_html`

Finally, FormMail responded to the browser.

If the form contained a `redirect` parameter, it sent a simple HTTP redirect header:

```perl
print "Location: http://example.com/thankyou.html\n\n";
```

Otherwise, it dynamically generated an HTML “Thank You” page listing all submitted form values — showing the user that their message was sent successfully.

---

## 🧩 Security and Legacy

FormMail was revolutionary — but also *infamous*.
Because it was so easy to install, many users neglected to configure the security arrays properly:

```perl
@referers = ('mydomain.com');
@recipients = ('^[\w\-\.]+\@mydomain\.com');
```

If left open, spammers discovered they could exploit the script to send arbitrary emails, using someone else’s server as a relay.

This led to widespread *FormMail abuse* in the early 2000s, with many web hosts banning it outright.

Later versions (like 1.91 and 1.92) added safeguards, including:

* Restricting valid recipients via domain pattern matching.
* Limiting reportable environment variables (`@valid_ENV`).
* Stripping newlines from email headers to prevent header injection.
* Sanitizing all HTML output (`clean_html()`).

Still, the damage to its reputation had been done — and FormMail gradually faded from use as PHP and server-side frameworks took over.

---

## 🧬 Why It Mattered

FormMail taught a generation of early web developers **how the web actually worked**:

* It exposed the relationship between HTML forms and CGI environment variables.
* It demonstrated server-side scripting before “backend” was a household term.
* It embodied the early web’s do-it-yourself spirit — readable, hackable, and educational.

Even with its flaws, FormMail remains a **historic artifact** — a bridge between the static web and the interactive internet.

---

## 🗃️ Example HTML Form

Here’s what a typical 1990s contact form looked like:

```html
<form method="POST" action="/cgi-bin/formmail.pl">
  <input type="hidden" name="recipient" value="info@example.com">
  <input type="hidden" name="subject" value="Website Contact">
  <input type="hidden" name="redirect" value="http://example.com/thanks.html">
  <input type="hidden" name="required" value="email,message">

  <p>Name: <input type="text" name="realname"></p>
  <p>Email: <input type="text" name="email"></p>
  <p>Message: <textarea name="message"></textarea></p>
  <p><input type="submit" value="Send"></p>
</form>
```

That’s all it took — one `.pl` script and a form tag.

---

## 🕰️ In Retrospect

FormMail is no longer used today — replaced by secure APIs, mail relays, and frameworks.
But it remains part of **the web’s genetic code** — one of the first scripts to make websites *listen*.

If you ever sent a message through a “Contact Us” form in the late 1990s —
you were speaking to Perl.

---

### 💬 “FormMail wasn’t just a script. It was the handshake between people and the early web.”

---

Would you like me to append a **timeline section** (1995–2005) showing how FormMail evolved, how spammers exploited it, and what modern replacements emerged (like PHPMailer, cgiemail, and Formspree)? It would make this readme an even more educational historical document.
