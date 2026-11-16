# June Klees, Ph.D

The personal website of June Klees, Ph.D: educator, academic coach, facilitator, podcaster, and speaker focused on applied civics. The site has a homepage, about and contact pages, and a library of content (articles, podcast episodes, talks, and events).

It's built with [Statamic 6](https://statamic.dev) (a flat-file CMS on [Laravel](https://laravel.com/docs)), with Antlers templates, Tailwind CSS 4, and Alpine.js. Content is stored as Markdown/YAML files in the repo, so there's no database to set up.

## Project layout

| Path | What's in it |
| --- | --- |
| `content/` | Entries, globals, navigation, and taxonomies (managed through the Control Panel) |
| `resources/views/` | Antlers templates (`layout.antlers.html` is the main layout) |
| `resources/blueprints/`, `resources/fieldsets/` | Field definitions for the Control Panel |
| `resources/forms/contact.yaml` | Contact form config, including where submission emails go |
| `resources/css/`, `resources/js/` | Front-end assets built by Vite |
| `public/assets/` | Uploaded images and files |
| `users/` | Control Panel user accounts |
| `storage/forms/` | Contact form submissions (gitignored) |

## Local development

**Requirements:** PHP 8.5+, Composer, and Node.js. On macOS, [Laravel Herd](https://herd.laravel.com) covers PHP and Composer. For other setups, see [Statamic's installation requirements](https://statamic.dev/getting-started/requirements).

1. Clone the repo and run the setup script:

   ```bash
   composer setup
   ```

   This installs PHP and npm dependencies, creates `.env` from `.env.example`, generates an app key, and builds the front-end assets.

2. Start the dev server and Vite:

   ```bash
   composer dev
   ```

   If you use Herd, link the folder ([Herd docs](https://herd.laravel.com/docs/macos/getting-started/sites)) and visit `http://juneklees.test` instead. Run `npm run dev` for hot reloading, and set `APP_URL` in `.env` to match your local URL.

3. Login to the Control Panel with the existing admin user.

### Contact form locally

By default `.env.example` sets `MAIL_MAILER=log`, but the contact form is set to send through Resend (`mailer: resend` in `resources/forms/contact.yaml`). To test sending for real, add `RESEND_API_KEY` to `.env`. Otherwise, check `storage/forms/contact/` for saved submissions.

The form uses Cloudflare Turnstile for spam protection. For local testing, use Cloudflare's [dummy test keys](https://developers.cloudflare.com/turnstile/troubleshooting/testing/) for `TURNSTILE_SITE_KEY` and `TURNSTILE_SECRET_KEY`.

### Useful docs

- [Statamic: Antlers templating](https://statamic.dev/antlers)
- [Statamic: Blueprints & fields](https://statamic.dev/blueprints)
- [Statamic: Forms](https://statamic.dev/forms)
- [Laravel: Configuration](https://laravel.com/docs/configuration)

## Production deployment

The site is deployed with [Coolify](https://coolify.io/docs), which builds the app from this repo (no Dockerfile, so Coolify's [Nixpacks](https://nixpacks.com/docs/providers/php) build is used) and redeploys on push to `main`. For general guidance, see Statamic's [deployment docs](https://statamic.dev/getting-started/deploying) and Laravel's [deployment docs](https://laravel.com/docs/deployment).

### Environment variables

Set these in Coolify's **Environment Variables** tab, starting from `.env.example`:

| Variable | Value |
| --- | --- |
| `APP_ENV` | `production` |
| `APP_DEBUG` | `false` |
| `APP_KEY` | Generate with `php artisan key:generate --show` |
| `APP_URL` | The live site URL (`https://...`) |
| `MAIL_MAILER` | `resend` |
| `RESEND_API_KEY` | From the [Resend dashboard](https://resend.com/docs/send-with-laravel) (the `juneklees.com` domain must be verified) |
| `TURNSTILE_SITE_KEY`, `TURNSTILE_SECRET_KEY` | From the Cloudflare Turnstile dashboard |
| `STATAMIC_LICENSE_KEY` | Only needed if Statamic Pro is enabled |

The app trusts all proxies (`bootstrap/app.php`) so HTTPS URLs are generated correctly behind Coolify's reverse proxy.

### Persistent storage

Each deploy creates a fresh container, so any files written at runtime are lost unless they're on a volume. In Coolify, go to **Configuration → Persistent Storage** and add a volume for form submissions:

- **Destination path:** `/app/storage/forms` (check the app's path with `pwd` in the container terminal if it differs)

Mount only `storage/forms`, not all of `storage/`, because Laravel needs the `storage/framework/` subfolders that ship with the build. If submissions fail to save after adding the volume, fix the ownership with a post-deployment command, for example `chown -R www-data:www-data /app/storage/forms`. Coolify doesn't back up file volumes, so back up this volume separately if the submissions matter.

### Editing content in production

Content, users, and uploaded assets live in the repo. **Changes made in the production Control Panel are lost on the next deploy** unless you either:

- make content changes locally and push them, or
- turn on [Statamic's Git integration](https://statamic.dev/git-automation) (`STATAMIC_GIT_ENABLED=true`) so Control Panel edits are committed back to the repo, or
- add persistent volumes for `content/`, `users/`, and `public/assets/`.
