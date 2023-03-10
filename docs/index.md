# Apple Auth

__Version:__ 0.0.1

Python library to implement Sign In with Apple in your Django backend.

### 🍎 Apple docs

From now on, some stuff is much better explained on the Apple docs, so when in doubt just check (if you haven't done so) the following documents:

- [Sign In With Apple](https://developer.apple.com/sign-in-with-apple/get-started/)
- [Request an authorization to the Sign in with Apple server](https://developer.apple.com/documentation/sign_in_with_apple/request_an_authorization_to_the_sign_in_with_apple_server)
- [Generate and validate tokens](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens)
- [Revoke tokens](https://developer.apple.com/documentation/sign_in_with_apple/revoke_tokens)

### 📝 Configuration

To start using the lib, some Apple Keys needs to be generated:

- `client_id` (string)
    - The identifier (App ID or Services ID) for your app. The identifier must not include your Team ID, to help prevent the possibility of exposing sensitive data to the end user.
- `client_secret` (string)
    - A secret JSON Web Token, generated by the developer, that uses the Sign in with Apple private key associated with your developer account. Authorization code and refresh token validation requests require this parameter.
- `team_id` (string)
    - Team ID of your developer account this can be found in your apple developer portal => identifier of your app => "App ID prefix".
- `key_id` (string)
    - The Key ID of the p8 file.

### 🚀 Usage

You can install the library directly from PYPI using pip:

```bash
pip install appleauth
```

Edit your settings.py file and update `INSTALLED_APPS` and `APPLE_CONFIG` with the appropriate keys generated via Apple Developer Portal:

```python
INSTALLED_APPS = [
        ...,
        "appleauth"
]

# Apple Config
APPLE_CONFIG = {
    "APPLE_KEY_ID": "",
    "APPLE_TEAM_ID": "",
    "APPLE_CLIENT_ID": "",
    "APPLE_PRIVATE_KEY": "",
    "APPLE_REDIRECT_URL": "{{BASE URL}}/auth/apple/token", # https://127.0.0.1:8000/auth/apple/token
    "APPLE_SCOPE": ["name", "email"],
    "RESPONSE_HANDLER_CLASS": "users.services.AppleSignInResponseHandler",
}
```

NOTE:

- In the above config, `APPLE_REDIRECT_URL` is an endpoint which serves as a proxy to redirect the response of Apple server authorization to the `redirect_url` passed as query param while generating Authorization URL.
- The response of authorization by Apple is a `POST request` where auth `code` and `state` is sent in request body. This endpoint converts the request body data to query params and send it to the redirect URL.

Create Response Handler Class and update path in `APPLE_CONFIG`, In this example we are considering it to be in `/users/services/AppleSignInResponseHandler`

```python
from appleauth.services import AppleAuthResponseHandler

class AppleSignInResponseHandler(AppleAuthResponseHandler):
    def handle_fetch_or_create_user(self, request, user_dict):
        email = user_dict.get("email", None)
        apple_id = user_dict.get("apple_id", None)

        # Implement a method to handle user creation
        user,  is_created = get_or_create_user(email, apple_id)
        context = {"is_created": is_created}

        return user, context

    def generate_response_json(self, user, extra_context):

        # Implement a serializer to serialize user data
        response = AuthUserSerializer(user, context=extra_context)

        return response.data
```

NOTE:

- `AuthUserSerializer` used in above ref. could be created as per app's functionality and contain fields which needs to be sent in response of authorization.
- `get_or_create_user` method used in above code ref. could be created as per app's functionality.

Update Routes:

```python
from rest_framework.routers import DefaultRouter
from appleauth.apis import AppleAuthViewset

default_router = DefaultRouter(trailing_slash=False)

default_router.register("auth/apple", AppleAuthViewset, basename="apple-auth")

urlpatterns = [...] + default_router.urls
```

### 🤖 Endpoints

- Provides following APIs:
    - [**Authorization URL API**](endpoints/#get-authorization-url)
        - It generates Apple's `authorization-url` used to redirect to Apple's Authorization Server to request consent from resource owner.
    - [**Authorize API**](endpoints/#authorize-user)
        - Exchange authorization code for access token.
        - Talk to resource server with access token and fetch user's profile information.
    - [**Authorize IOS Token API**](endpoints/#authorize-ios-token)
        - Verifies an ID Token issued by Apple's authorization server.
        - Fetch user details from decoded token.

**NOTE:** This documentation changes frequently, checkout the [changelog](changelog.md) for detailed breaking changes and features added.

Write your documentation using **Markdown** in `docs/` folder. Need help? Read mkdocs [documentation][mkdocs].

[mkdocs]: http://www.mkdocs.org/user-guide/writing-your-docs/
