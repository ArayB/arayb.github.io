# Capybara Cheat Sheet

Check a link or button is not present:

```
expect(page).to_not have_selector(:link_or_button, "Button Text")
```