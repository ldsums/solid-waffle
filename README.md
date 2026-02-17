# solid-waffle

```
# Use a lightweight Debian-based image
FROM debian:bullseye-slim

# Install dependencies and Chrome
RUN apt-get update && apt-get install -y \
    wget \
    gnupg \
    ca-certificates \
    apt-transport-https \
    --no-install-recommends \
    && wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google.list \
    && apt-get update \
    && apt-get install -y google-chrome-stable --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# Run as a non-privileged user (Security Best Practice)
RUN groupadd -r chrome && useradd -r -g chrome -G audio,video chrome \
    && mkdir -p /home/chrome && chown -R chrome:chrome /home/chrome
USER chrome

ENTRYPOINT ["google-chrome", "--headless", "--disable-gpu", "--no-sandbox"]
```
```
FROM node:20-slim

# --- LAYER 1: System Utilities ---
# We install these once and they rarely change
RUN apt-get update && apt-get install -y \
    wget gnupg ca-certificates jq \
    --no-install-recommends

# --- LAYER 2: Google Chrome ---
# Keeping this separate allows Docker to cache the browser specifically
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/googlechrome-linux-keyring.gpg \
    && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/googlechrome-linux-keyring.gpg] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list \
    && apt-get update \
    && apt-get install -y google-chrome-stable --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# --- LAYER 3: Global NPM Packages ---
RUN npm install -g lighthouse

# --- LAYER 4: Application Files ---
WORKDIR /app
COPY chrome-flags.json lighthouse-config.js entrypoint.sh ./
RUN chmod +x /app/entrypoint.sh

ENTRYPOINT ["/app/entrypoint.sh"]
```
