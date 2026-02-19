# Testing

## Run the tests
`python manage.py test`

## Test classes:
- `SimpleTestCase`
  - When no database is needed
- `TestCase`
  - When a database is needed
- `TransactionTestCase`
  - When database transactions are being tested
- `LiveServerTestCase`
  - When a browser window is needed

## Assertions:
- `assertEqual(actual, expected)`
- `assertContains(response, expected_text)`
- `assertTemplateUsed(response, template_name)`
- `assertRedirects(response, expected_url, fetch_redirect_response = True)`
  - Set `fetch_redirect_response=False` when testing redirections from a mixin
