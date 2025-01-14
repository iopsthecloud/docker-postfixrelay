# docker-postfixrelay

[![CI/CD](https://github.com/loganmarchione/docker-postfixrelay/actions/workflows/main.yml/badge.svg)](https://github.com/loganmarchione/docker-postfixrelay/actions/workflows/main.yml)
[![Docker Image Size (latest semver)](https://img.shields.io/docker/image-size/loganmarchione/docker-postfixrelay)](https://hub.docker.com/r/loganmarchione/docker-postfixrelay)

Runs Postfix (as a relay) in Docker
  - Source code: [GitHub](https://github.com/loganmarchione/docker-postfixrelay)
  - Docker container: [Docker Hub](https://hub.docker.com/r/loganmarchione/docker-postfixrelay)
  - Image base: [Alpine Linux](https://hub.docker.com/_/alpine/)
  - Init system: [dumb-init](https://github.com/Yelp/dumb-init)
  - Application: [Postfix](http://www.postfix.org/)
  - Architecture: `linux/amd64,linux/arm64,linux/arm/v7`

## Explanation

  - This runs Postfix (as a relay) in Docker.
  - Most home ISPs block port 25, so outbound emails must be relayed through an external SMTP server (e.g., Gmail).
  - This container acts as a single collections point for devices needing to send email.
  - ⚠️ Postfix acts as an open relay. As such, this is not meant to be run on the internet, only on a trusted internal network! ⚠️

```
        Internal (LAN) network                                        Public internet

------------------
|                |
| Device sending |                            |   |
| email alert    | -------------              |   |
|                |             |              |   |
------------------             |              |   |
                               |              | F |
------------------             v              | i |
|                |       ------------------   | r |    -----------------------------       -------------------
| Device sending |       |                |   | e |    |                           |       |                 |
| email alert    | ----> | This container | --| w |--> | SMTP server (e.g., Gmail) | ----> | Recipient email |
|                |       |                |   | a |    |                           |       |                 |
------------------       ------------------   | l |    -----------------------------       -------------------
                               ^              | l |
------------------             |              |   |
|                |             |              |   |
| Device sending |             |              |   |
| email alert    | -------------              |   |
|                |
------------------
```

## Requirements

  - You must already have a account on an external SMTP server (e.g., Gmail, AWS SES, etc...).
  - Your external SMTP server must be using encryption (i.e., plaintext is not allowed)

## Docker image information

### Docker image tags
  - `latest`: Latest version
  - `X.X.X`: [Semantic version](https://semver.org/) (use if you want to stick on a specific version)

### Environment variables
| Variable    | Required?                 | Definition                                  | Example                    | Comments                                                     |
|-------------|---------------------------|---------------------------------------------|----------------------------|--------------------------------------------------------------|
| TZ          | Yes                       | Timezone                                    | America/New_York           | https://en.wikipedia.org/wiki/List_of_tz_database_time_zones |
| RELAY_HOST  | Yes                       | Public SMTP server to use                   | smtp.gmail.com             |                                                              |
| RELAY_PORT  | Yes                       | Public SMTP port to use                     | 587                        |                                                              |
| RELAY_USER  | No                        | Address to login to $RELAY_HOST             | SMTP username              |                                                              |
| RELAY_PASS  | No                        | Password to login to $RELAY_HOST            | SMTP password              | If using Gmail 2FA, you will need to setup an app password   |
| TEST_EMAIL  | No                        | Address to receive test email               | receive_address@domain.com | If not set, test email will **not** be sent                  |
| MYORIGIN    | No                        | Domain of the "from" address                | domain.com                 | Needed for things like AWS SES where the domain must be set  |
| MYNETWORKS  | No (default: 0.0.0.0/0)   | Networks that Postfix will forward mail for | 1.2.3.4/24, 5.6.7.8/24     | Single or multiple trusted network separated with a comma |
| SMTP_TLS_SECURITY_LEVEL  | No                        | Sepcify TLS configuration $RELAY_HOST             | may              | default set to "encrypt"                                                             |
| SMTP_SASL_AUTH_ENABLE  | No                        | Enable or disabled SASL AUTH             | no              | default set to "yes"                                                             |


### Ports
| Port on host              | Port in container | Comments            |
|---------------------------|-------------------|---------------------|
| Choose at your discretion | 25                | Postfix SMTP server |

### Volumes
| Volume on host            | Volume in container | Comments                           |
|---------------------------|---------------------|------------------------------------|
| Choose at your discretion | /var/spool/postfix  | Used to store Postfix's mail spool |

### Example usage
Below is an example docker-compose.yml file.
```
version: '3'
services:
  postfixrelay:
    container_name: postfixrelay
    restart: unless-stopped
    environment:
      - TZ=America/New_York
      - RELAY_HOST=smtp.gmail.com
      - RELAY_PORT=587
      - RELAY_USER=your_email_here@gmail.com
      - RELAY_PASS=your_password_here
      - TEST_EMAIL=test_email@domain.com
      - MYORIGIN=domain.com
      - MYNETWORKS=1.2.3.4/24
    networks:
      - postfixrelay
    ports:
      - '25:25'
    volumes:
      - 'postfixrelay_data:/var/spool/postfix'
    image: loganmarchione/docker-postfixrelay:latest

networks:
  postfixrelay:

volumes:
  postfixrelay_data:
    driver: local
```

## TODO
- [x] ~~Add a [healthcheck](https://docs.docker.com/engine/reference/builder/#healthcheck)~~
- [ ] Add TLS support for SMTPD and listen on 587
