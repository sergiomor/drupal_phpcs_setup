# Drupal Coding Standards & PHPCS Setup

This guide helps you configure and enforce **Drupal coding standards** in your development environment using `phpcs`, integrates it with **Visual Studio Code**, and sets up a **Git pre-commit hook** to catch issues before commits.

## 📦 Install PHPCS
```bash
composer require --dev squizlabs/php_codesniffer
./vendor/bin/phpcs --version
```

## 🎯 Install Drupal Coding Standards
```bash
composer require --dev drupal/coder
./vendor/bin/phpcs --config-set installed_paths vendor/drupal/coder/coder_sniffer
./vendor/bin/phpcs --config-set default_standard Drupal
./vendor/bin/phpcs --standard=Drupal web/modules/custom
composer require --dev slevomat/coding-standard
```

## 🧠 Set up PHPCS in Visual Studio Code

Install the VS Code extension: **PHP Sniffer & Beautifier** (`ikappas.phpcs`)

Create or update `.vscode/settings.json`:
```json
{
  "phpcs.enable": true,
  "phpcs.executablePath": "./vendor/bin/phpcs",
  "phpcs.standard": "Drupal",
  "php.validate.executablePath": "/usr/bin/php"
}
```

## 🏃 Run PHPCS manually
```bash
./vendor/bin/phpcs --standard=Drupal path/to/your/file.php
./vendor/bin/phpcbf --standard=Drupal path/to/your/file.php
```

## 🔒 Add a Git Pre-commit Hook
```bash
touch .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

Paste into `.git/hooks/pre-commit`:
```bash
#!/bin/sh

PHPCS="./vendor/bin/phpcs"
STANDARD="Drupal"
FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.php$')

if [ "$FILES" = "" ]; then
  exit 0
fi

echo "🔍 Running PHPCS on staged PHP files..."

PASS=true

for FILE in $FILES; do
  $PHPCS --standard=$STANDARD "$FILE"

  if [ $? -ne 0 ]; then
    PASS=false
  fi
done

if ! $PASS; then
  echo ""
  echo "❌ Coding standards violations found."
  echo "💡 Run: ./vendor/bin/phpcbf --standard=Drupal <file> to auto-fix."
  exit 1
fi

echo "✅ All staged PHP files passed Drupal coding standards."
exit 0
```

## 💡 Tips

- Use `phpcbf` to automatically fix most issues.
- Run `phpcs` before committing large changes to avoid delays.
- Use `DrupalPractice` as the standard for deeper architectural reviews.
- Use `.editorconfig` to help standardize spacing, tabs, and line endings.
