<div align="center">

# 🔒 Restrict Preview for Tutor LMS

### Turn free preview lessons into registered users.

<p>
  <img alt="WordPress" src="https://img.shields.io/badge/WordPress-5.8%2B-21759B?style=for-the-badge&logo=wordpress&logoColor=white">
  <img alt="PHP" src="https://img.shields.io/badge/PHP-7.4%2B-777BB4?style=for-the-badge&logo=php&logoColor=white">
  <img alt="Tutor LMS" src="https://img.shields.io/badge/Tutor%20LMS-required-1E2A78?style=for-the-badge">
  <img alt="License" src="https://img.shields.io/badge/License-GPL--2.0%2B-3DA639?style=for-the-badge&logo=gnu&logoColor=white">
</p>

<p>
  <a href="../../actions/workflows/plugin-check.yml">
    <img alt="Plugin Check" src="https://img.shields.io/github/actions/workflow/status/salma25128/restrict-preview-for-tutor-lms/plugin-check.yml?branch=main&style=flat-square&label=plugin%20check">
  </a>
  <a href="../../releases">
    <img alt="Release" src="https://img.shields.io/github/v/release/salma25128/restrict-preview-for-tutor-lms?style=flat-square&color=1E2A78&include_prereleases&sort=semver">
  </a>
  <a href="../../commits/main">
    <img alt="Last commit" src="https://img.shields.io/github/last-commit/salma25128/restrict-preview-for-tutor-lms?style=flat-square">
  </a>
  <a href="../../issues">
    <img alt="Issues" src="https://img.shields.io/github/issues/salma25128/restrict-preview-for-tutor-lms?style=flat-square">
  </a>
</p>

<p>
  <a href="#problem">Problem</a> ·
  <a href="#how-it-works">How it works</a> ·
  <a href="#install">Install</a> ·
  <a href="#settings">Settings</a> ·
  <a href="#developers">Developers</a> ·
  <a href="#releasing">Releasing</a>
</p>

</div>

---

<a id="problem"></a>

## 🎯 The problem

Tutor LMS lets you mark lessons as **free previews**, so anyone can watch them. Great for marketing — but visitors consume your best sample content and leave without ever telling you who they are.

**Restrict Preview for Tutor LMS** asks for a free account first. Guests get a tabbed Log In / Sign Up form in place of the lesson, and land on the exact lesson they clicked the moment they're in.

<table>
<tr>
<th width="50%">❌ Without the plugin</th>
<th width="50%">✅ With the plugin</th>
</tr>
<tr>
<td>Anonymous visitor watches your preview lessons and leaves. No email, no account, no follow-up.</td>
<td>Visitor creates a free account to watch the same lesson. You gain a lead; they lose nothing.</td>
</tr>
<tr>
<td>Preview URLs get shared and indexed, bypassing your funnel entirely.</td>
<td>Direct URLs, crawlers and JS-disabled browsers all hit the same server-side gate.</td>
</tr>
<tr>
<td>"Log in" sends people to a separate page and loses their place.</td>
<td>Login and signup happen inline, then return them to the lesson.</td>
</tr>
</table>

---

## ⚡ Features

<table>
<tr>
<td width="33%" valign="top">

### 🔐 Server-side gate
Enforced in a `the_content` filter before the page is sent. Not a JavaScript overlay — direct URLs and crawlers get the same treatment.

</td>
<td width="33%" valign="top">

### 🗂️ Tabbed auth
Log In and Sign Up in one popup, built from Tutor's own shortcodes. No reimplemented authentication.

</td>
<td width="33%" valign="top">

### ↩️ Smart return
Uses Tutor's supported `redirect_to` field to land the visitor on the lesson they actually clicked.

</td>
</tr>
<tr>
<td valign="top">

### 🏷️ Curriculum badges
"Free Preview" on each preview lesson, plus a free-lesson count per section. Matched by URL, not by guessing markup.

</td>
<td valign="top">

### 🔖 Auto field labels
Fields injected by other plugins (a phone number, say) get a visible title instead of an unlabelled box.

</td>
<td valign="top">

### 🌍 Translation ready
Full `.pot`, RTL-safe CSS using logical properties, and text defaults translated at read time — never frozen into the database.

</td>
</tr>
</table>

---

<a id="how-it-works"></a>

## 🧭 How it works

```mermaid
flowchart TD
    A[Visitor opens a lesson] --> B{Marked as<br/>free preview?}
    B -- No --> C[Untouched —<br/>Tutor's enrolment rules apply]
    B -- Yes --> D{Course marked<br/>public in Tutor?}
    D -- Yes --> C
    D -- No --> E{Logged in?}
    E -- Yes --> F[Full access]
    E -- No --> G[Show Log In / Sign Up]
    G --> H[Submit form]
    H --> I[Redirect to the<br/>requested lesson]
    I --> F

    style C fill:#ECFDF5,stroke:#047857,color:#0F172A
    style F fill:#ECFDF5,stroke:#047857,color:#0F172A
    style G fill:#FEF3C7,stroke:#92400E,color:#0F172A
```

### Who is affected

| Visitor | Preview lessons | Other lessons |
|---|:---:|:---:|
| 👤 **Guest** | 🔒 Asked to sign in | 🔒 Tutor's rules |
| ✅ **Registered, not enrolled** | ✅ Full access | 🔒 Tutor's rules |
| 🎓 **Enrolled student** | ✅ Full access | ✅ Full access |
| 🧑‍🏫 **Instructor** | ✅ Full access | ✅ Tutor's rules |
| 🛠️ **Administrator** | ✅ Full access | ✅ Full access |

> [!NOTE]
> Enrolment is evaluated **per course** by Tutor, so a student enrolled in one course gains nothing extra in another. Being enrolled elsewhere has no effect on this plugin either — any logged-in user is simply treated as registered.

> [!IMPORTANT]
> Lessons in a course marked **public** in Tutor are deliberately left alone. A public course is an explicit decision to open everything to everyone, and gating there would override the site owner.

<details>
<summary><b>🔍 Implementation detail — how a preview lesson is detected</b></summary>

<br>

A lesson counts as a preview when Tutor's own `_is_preview` post meta is truthy:

```php
public static function is_preview( $lesson_id ) {
    return (bool) (int) get_post_meta( $lesson_id, '_is_preview', true );
}
```

This mirrors the exact check in Tutor's `CourseModel::has_course_content_access()`. The meta is stored as an **integer**, so comparing it against the string `'yes'` silently matches nothing — a subtle failure mode where every lesson looks un-gated and the plugin appears to do nothing at all.

Curriculum badges are matched to lessons by **exact permalink**, not by scanning rendered text for the word "Free". Many themes and page builders never print that word, so text matching finds nothing on perfectly ordinary courses.

</details>

---

<a id="install"></a>

## 📦 Installation

**Requirements**

| | |
|---|---|
| WordPress | `5.8+` |
| PHP | `7.4+` |
| Tutor LMS | Required — a notice appears and the plugin stays inert without it |

1. Download the latest zip from [releases](../../releases).
2. **Plugins → Add New → Upload Plugin**.
3. Activate, then open **Settings → Restrict Preview**.

> [!TIP]
> The **Sign Up** tab needs *Anyone can register* enabled under **Settings → General**. Tutor's registration handler refuses to create accounts while that's off, so the tab hides itself rather than becoming a dead end. If Sign Up is missing, that's why.

---

<a id="settings"></a>

## ⚙️ Settings

Everything lives under **Settings → Restrict Preview**.

| Group | Options |
|---|---|
| **Behaviour** | Restrict preview lessons · Show curriculum badges · Show the Sign Up tab |
| **Wording** | Badge label · Curriculum badge · Popup heading · Popup message · Lesson page message |
| **Appearance** | Accent colour · Account page URL (fallback) |

<details>
<summary><b>💡 Why cleared fields fall back to the default</b></summary>

<br>

Text defaults are translated at **read** time and never written to the database:

```php
self::$cache = wp_parse_args( $stored, self::defaults() );
```

Empty stored values are dropped before that merge. Two consequences worth knowing:

1. Clearing a field in the admin **restores the default** rather than blanking the text.
2. Changing the site language changes the wording, instead of leaving the original language frozen in the options table forever.

</details>

---

<a id="developers"></a>

## 🧑‍💻 Developers

### Filters

```php
/**
 * Widen or narrow who may view preview lessons.
 * Default: any logged-in user.
 */
add_filter( 'rptl_user_may_view_preview', function ( $allowed ) {
    // Example: require a specific capability instead.
    return current_user_can( 'read' );
} );

/**
 * Force assets and the popup to load somewhere unusual —
 * a page builder template or a custom archive rendering a curriculum.
 */
add_filter( 'rptl_is_relevant_context', function ( $relevant ) {
    return $relevant || is_page( 'my-custom-curriculum' );
} );
```

### Architecture

```
restrict-preview-for-tutor-lms/
├── restrict-preview-for-tutor-lms.php   Bootstrap · headers · Tutor dependency check
├── includes/
│   ├── class-rptl-plugin.php            Wires the pieces together
│   ├── class-rptl-settings.php          Options · sanitisation · admin screen
│   ├── class-rptl-lessons.php           Preview detection · curriculum data · context
│   ├── class-rptl-access.php            The gate · post-auth redirect
│   ├── class-rptl-auth-ui.php           Locked card · popup · tabbed form
│   └── class-rptl-assets.php            Enqueue · localised data
├── assets/css/restrict-preview.css      Front-end styles
├── assets/js/restrict-preview.js        Popup · tabs · badges · labels
├── languages/*.pot                      36 translatable strings
└── uninstall.php                        Multisite-aware option cleanup
```

<details>
<summary><b>🧩 Design decisions</b></summary>

<br>

| Decision | Reasoning |
|---|---|
| Embed Tutor's shortcodes rather than build forms | Validation, password handling and account creation stay in Tutor, where they're maintained and audited |
| Gate in `the_content`, not via JS | A JavaScript overlay is bypassed by disabling JS or fetching the URL directly |
| Badges matched by permalink | Container class names differ across Tutor versions, themes and page builders; guessing them fails silently |
| Popup and assets share one context check | Rendering the popup where the stylesheet hasn't loaded would drop unstyled, inert forms into the footer |
| `redirect_to` primary, cookie as fallback | Tutor honours `redirect_to` natively; the cookie only covers flows that strip it, and is validated to a real lesson URL on the same site |

</details>

---

<a id="releasing"></a>

## 🚀 Releasing

Deployment to WordPress.org is automated, but **only works after the plugin is approved** — approval is what creates the SVN repository the workflow pushes into.

<details open>
<summary><b>One-time setup</b></summary>

<br>

1. Add repository secrets under **Settings → Secrets and variables → Actions**:

   | Secret | Value |
   |---|---|
   | `SVN_USERNAME` | Your WordPress.org username |
   | `SVN_PASSWORD` | Your WordPress.org password |

2. Add the directory graphics to [`.wordpress-org/`](.wordpress-org) — see that folder's README for exact filenames. They live there rather than in `assets/`, which the plugin's own CSS and JS already occupy.

</details>

<details>
<summary><b>Publishing a version</b></summary>

<br>

```bash
# 1. Bump the version in BOTH places — they must match
#    restrict-preview-for-tutor-lms.php   → " * Version:  1.0.1"
#    readme.txt                           → "Stable tag: 1.0.1"

# 2. Add a == Changelog == entry in readme.txt

# 3. Tag and publish a GitHub release with the bare version number
git tag 1.0.1 && git push origin 1.0.1
```

The workflow verifies the release tag, the plugin header and the stable tag all agree **before** deploying, and fails the run if they don't. A mismatch there is the usual way a release ends up published but unserved by the directory.

</details>

### Workflows

| Workflow | Trigger | Needs secrets |
|---|---|:---:|
| [`plugin-check.yml`](.github/workflows/plugin-check.yml) | Push · PR | No |
| [`deploy.yml`](.github/workflows/deploy.yml) | Published release | Yes |

`plugin-check.yml` runs WordPress's official Plugin Check plus PHP lint on 7.4, 8.2 and 8.3 — the same automated checks the review team runs, so problems surface on a pull request instead of in a rejection email. Files excluded from the deployed package are listed in [`.distignore`](.distignore).

---

## 📄 License

[GPL-2.0-or-later](LICENSE.txt) · Built for [Tutor LMS](https://wordpress.org/plugins/tutor/) by Themeum, which is not affiliated with this plugin.

<div align="center">
<br>
<sub>Built by <a href="https://github.com/salma25128">Salma Basuony</a></sub>
</div>