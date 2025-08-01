# Best Practice Reminder
Keep your .env file outside the project folder on the VPS:
```
/home/deploy/.env.fbaserver
```
And your docker-compose.yml should point to it:
```
env_file:
  - /home/deploy/.env.fbaserver
```