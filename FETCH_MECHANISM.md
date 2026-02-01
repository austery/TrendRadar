# TrendRadar Data Fetching Mechanism

This document explains how TrendRadar fetches data from various sources, including domestic Chinese platforms and RSS feeds.

## Data Sources

TrendRadar primarily fetches data from two types of sources:

1.  **Platform Hot Lists** (e.g., Weibo, Zhihu, Toutiao, Baidu)
2.  **RSS Feeds**

### 1. Platform Hot Lists

For hot lists from platforms like Weibo, Zhihu, and others, TrendRadar uses an external API provided by the **newsnow** project.

*   **API Endpoint**: `https://newsnow.busiyi.world/api/s`
*   **Method**: `GET`
*   **Parameter**: `id` (The platform ID, e.g., `weibo`, `zhihu`)
*   **Response Format**: JSON containing the list of trending items.
*   **Implementation**:
    *   The logic resides in `trendradar/crawler/fetcher.py`.
    *   The `DataFetcher` class handles the HTTP requests.
    *   It does **not** scrape the target platforms directly within this codebase. It delegates the fetching to the `newsnow` API service.

**Example Request:**
```http
GET https://newsnow.busiyi.world/api/s?id=weibo&latest
```

### 2. RSS Feeds

For RSS subscriptions, TrendRadar fetches the data directly from the source.

*   **Method**: Direct HTTP `GET` request to the RSS feed URL.
*   **Parsing**: Uses the `feedparser` library (if available) or a fallback JSON Feed parser.
*   **Implementation**:
    *   The logic resides in `trendradar/crawler/rss/fetcher.py`.
    *   The `RSSFetcher` class manages the fetching of configured feeds.
    *   The `RSSParser` class in `trendradar/crawler/rss/parser.py` handles the parsing of the XML/JSON response.

## Technical Analysis

### API vs. Reverse Engineering

The user asked whether the system uses an API or "reverse cracking" (reverse engineering).

*   **From TrendRadar's Perspective**: It uses **APIs**.
    *   It uses the public-facing API of the `newsnow` service.
    *   It uses standard HTTP/RSS protocols.
    *   It does **not** contain code to reverse engineer the frontend or internal APIs of platforms like Weibo or Zhihu.

*   **Underlying Mechanism (Inferred)**:
    *   The `newsnow` service (the upstream provider) likely uses techniques such as **reverse engineering** of internal mobile APIs or **web scraping** to aggregate data from these platforms, as most of them do not offer official public APIs for "hot lists".
    *   However, this complexity is abstracted away from TrendRadar.

### Domestic Data Sources

*   **Yes**, the data originates from domestic Chinese platforms (Weibo, Baidu, Toutiao, etc.).
*   TrendRadar accesses this data via the `newsnow` API, which aggregates these domestic sources.
