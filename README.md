GitHub → PostHog Sync Script

This script pulls events (issues, PRs, pushes, comments, etc.) from a GitHub repository and sends them into PostHog as events.

## Requirements

Python 3.9+

A GitHub Personal Access Token

A PostHog API Key

requests Python library

##Install dependencies
pip install requests

## Environmental Variables
Set these before running:
export GITHUB_TOKEN="your_github_token"
export POSTHOG_API_KEY="your_posthog_key"
export POSTHOG_HOST="https://app.posthog.com" # or self-hosted url

## Run the Script:
python sync.py owner repo

## Example
python sync.py openai gpt

## sync.py
#!/usr/bin/env python3
import os
import sys
import requests


GITHUB_API = "https://api.github.com"
POSTHOG_ENDPOINT = "/capture/"




def get_github_events(owner, repo, token):
url = f"{GITHUB_API}/repos/{owner}/{repo}/events"
headers = {"Authorization": f"Bearer {token}"}
resp = requests.get(url, headers=headers)
resp.raise_for_status()
return resp.json()




def send_to_posthog(api_key, host, event_name, properties):
url = f"{host}{POSTHOG_ENDPOINT}"
payload = {
"api_key": api_key,
"event": event_name,
"properties": properties,
}
resp = requests.post(url, json=payload)
resp.raise_for_status()
return resp.json()




def main():
if len(sys.argv) < 3:
print("Usage: python sync.py <owner> <repo>")
sys.exit(1)


owner = sys.argv[1]
repo = sys.argv[2]


github_token = os.getenv("GITHUB_TOKEN")
posthog_key = os.getenv("POSTHOG_API_KEY")
posthog_host = os.getenv("POSTHOG_HOST", "https://app.posthog.com")


if not github_token or not posthog_key:
print("Missing environment variables. Please set GITHUB_TOKEN and POSTHOG_API_KEY.")
sys.exit(1)


print(f"Fetching GitHub events for {owner}/{repo}...")
events = get_github_events(owner, repo, github_token)


for ev in events:
event_type = ev.get("type", "GitHubEvent")
properties = {
"repo": f"{owner}/{repo}",
"id": ev.get("id"),
"actor": ev.get("actor", {}).get("login"),
"created_at": ev.get("created_at"),
"raw": ev,
}


print(f"Sending event: {event_type} ...")
send_to_posthog(posthog_key, posthog_host, event_type, properties)


print("Sync complete!")




if __name__ == "__main__":
main()
