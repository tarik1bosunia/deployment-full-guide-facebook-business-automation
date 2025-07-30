
# stack summery

| Component    | Role                           |
| ------------ | ------------------------------ |
| Django       | Web API + WebSocket handler    |
| Daphne       | ASGI HTTP/WebSocket server     |
| Celery       | Background tasks               |
| Redis        | Celery broker + Channels layer |
| PostgreSQL   | Relational DB (in Docker)      |
| Nginx (host) | Reverse proxy + WebSocket      |


### Docker runs:

- Django + Gunicorn
- Celery worker
- Redis
- PostgreSQL

### Host OS runs:
- Nginx
- certbot for HTTPS (if needed)

