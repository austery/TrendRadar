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

## The "Magic" Behind the API: NewsNow

You might wonder why a single API endpoint (`https://newsnow.busiyi.world/api/s`) can return data from so many different platforms (Weibo, Zhihu, etc.) and if there is "hidden code".

The answer is **yes**, the heavy lifting is done by the upstream server, but the code is **not hidden**—it is open source.

*   **Upstream Project**: [NewsNow (ourongxing/newsnow)](https://github.com/ourongxing/newsnow)
*   **How it works**:
    *   **Server-Side Scraping**: The `newsnow` server runs independent scrapers for each platform.
    *   **Reverse Engineering**: The authors of `newsnow` have analyzed the internal APIs or HTML structures of platforms like Weibo and Zhihu to extract the "Hot List" data.
    *   **Aggregation**: The server standardizes this data into a unified JSON format.
    *   **TrendRadar's Role**: TrendRadar acts as a *client* to this server, fetching the pre-processed data to perform its own analysis (keyword filtering, trend tracking, AI analysis).

So, the "reverse cracking" logic exists, but it resides in the [NewsNow repository](https://github.com/ourongxing/newsnow), not in TrendRadar. This architecture decouples the complexity of maintaining dozens of scrapers from the logic of analyzing and pushing news.

## Technical Analysis

### API vs. Reverse Engineering

*   **From TrendRadar's Perspective**: It uses **APIs**.
    *   It uses the public-facing API of the `newsnow` service.
    *   It uses standard HTTP/RSS protocols.

*   **From the Data Source Perspective**: It involves **Reverse Engineering/Scraping**.
    *   The `newsnow` service uses techniques such as reverse engineering of internal mobile APIs or web scraping to aggregate data from these platforms.
