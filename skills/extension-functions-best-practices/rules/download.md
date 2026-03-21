# Download Features Implementation Guide

## Common Feature Types

- **Batch Download**: Multiple file downloads
- **Download Management**: Queue, resume, multi-threading
- **Resource Sniffing**: Auto-detect downloadable resources

## Core APIs and Implementation

### Resource Sniffing (MV2)

```javascript
// MV2 webRequest API
chrome.webRequest.onBeforeRequest.addListener(
  (details) => {
    if (isDownloadableResource(details.url, details.type)) {
      captureResource(details);
    }
  },
  { urls: ['<all_urls>'] },
  ['requestBody']
);

function isDownloadableResource(url, type) {
  const videoExts = ['.mp4', '.webm', '.m3u8', '.mpd'];
  const audioExts = ['.mp3', '.m4a', '.wav', '.ogg'];
  
  return videoExts.some(ext => url.includes(ext)) ||
         audioExts.some(ext => url.includes(ext)) ||
         type === 'media';
}
```

### Resource Sniffing (MV3)

```javascript
// Use declarativeNetRequest for known patterns
// Or content script to intercept fetch/XHR

// content_script.js
const originalFetch = window.fetch;
window.fetch = async function(...args) {
  const response = await originalFetch.apply(this, args);
  
  // Capture media responses
  const contentType = response.headers.get('content-type');
  if (contentType && contentType.includes('video')) {
    chrome.runtime.sendMessage({
      type: 'CAPTURED_MEDIA',
      url: args[0]
    });
  }
  
  return response;
};
```

### Download with Custom Headers

```javascript
chrome.downloads.download({
  url: resourceUrl,
  filename: 'video.mp4',
  headers: [
    { name: 'Referer', value: pageUrl },
    { name: 'User-Agent', value: navigator.userAgent }
  ],
  saveAs: false
});
```

### External Downloader Integration

```javascript
// URL Protocol scheme
function openWithExternalDownloader(url, referer) {
  const command = `mydownloader://download?url=${encodeURIComponent(url)}&referer=${encodeURIComponent(referer)}`;
  window.open(command);
}

// Native Messaging (requires native host)
async function downloadWithNativeHost(url) {
  const port = chrome.runtime.connectNative('com.mycompany.downloader');
  port.postMessage({ action: 'download', url });
  
  port.onMessage.addListener((response) => {
    console.log('Download status:', response);
  });
}
```

### Download Progress Tracking

```javascript
chrome.downloads.onCreated.addListener((downloadItem) => {
  console.log('Download started:', downloadItem.id);
});

chrome.downloads.onChanged.addListener((delta) => {
  if (delta.state) {
    console.log('Download state changed:', delta.state.current);
  }
  if (delta.bytesReceived) {
    console.log('Progress:', delta.bytesReceived.current);
  }
});
```

## Permissions Required

- `downloads` - Manage downloads
- `webRequest` (MV2) / `declarativeNetRequest` (MV3) - Sniff resources
- `nativeMessaging` - Integrate external downloaders

## Best Practices

1. **Memory Management**: Stream large files, don't buffer in memory
2. **CORS/Referer**: Pass correct headers for hotlink-protected resources
3. **File Naming**: Sanitize filenames, preserve original extensions
4. **Conflict Handling**: Handle duplicate filenames
5. **Progress UI**: Show download progress for large files
6. **Error Recovery**: Implement retry logic for failed downloads

## Reference Projects

| Project | Features | GitHub |
|---------|----------|--------|
| Cat Catch | M3U8/MPD sniffing, external integration | https://github.com/xifangczy/cat-catch |
| Stream Detector | Cookie export, stream detection | https://github.com/54ac/stream-detector |
| Turbo Download Manager | Multi-threading, resume | https://github.com/inbasic/turbo-download-manager-v2 |

## External Downloaders

- **N_m3u8DL-RE**: https://github.com/nilaoda/N_m3u8DL-RE
- **yt-dlp**: https://github.com/yt-dlp/yt-dlp
- **aria2**: https://github.com/aria2/aria2
