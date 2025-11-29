# A1 Language Certification Paper

## Running Locally with Jekyll

### Prerequisites

1. **Install Ruby** (if not already installed):
   - Download RubyInstaller for Windows: https://rubyinstaller.org/
   - Choose Ruby+Devkit version (recommended)
   - Run the installer and follow the prompts
   - Restart your terminal after installation

2. **Install Jekyll and dependencies**:
   ```bash
   gem install bundler
   bundle install
   ```

3. **Run Jekyll locally**:
   ```bash
   bundle exec jekyll serve
   ```

4. **View your site**:
   - Open your browser to: http://localhost:4000
   - The site will auto-reload when you make changes

### Alternative: Quick Start (if Ruby is installed)

```bash
gem install jekyll bundler
bundle install
bundle exec jekyll serve
```

## GitHub Pages

GitHub Pages automatically processes Jekyll sites. Just push your changes to the repository and GitHub will build and deploy your site.

If Jekyll isn't working on GitHub Pages, check:
- Repository Settings → Pages → Make sure "Source" is set to your branch
- Ensure `_config.yml` exists
- Check for any build errors in the Actions tab


