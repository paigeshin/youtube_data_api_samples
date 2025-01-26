```jsx
const APIKey = process.env.API_KEY;
const axios = require("axios");
const fs = require("fs");
const path = require("path");

// 동영상 정보 가져오기
async function getVideoDetails(videoId) {
  try {
    const response = await axios.get(
      "https://www.googleapis.com/youtube/v3/videos",
      {
        params: {
          part: "snippet,statistics",
          id: videoId,
          key: APIKey,
        },
      }
    );

    const video = response.data.items[0]; // 첫 번째 결과
    if (video) {
      console.log(video);
      const filePath = path.join(__dirname, "video.json");
      fs.writeFileSync(filePath, JSON.stringify(video, null, 2));
    } else {
      console.log("❌ 해당 비디오를 찾을 수 없습니다.");
    }
  } catch (error) {
    console.error(
      "오류 발생:",
      error.response ? error.response.data : error.message
    );
  }
}

// 동영상 정보 가져오기
async function getVideoReplies(videoId) {
  try {
    const response = await axios.get(
      "https://www.googleapis.com/youtube/v3/commentThreads",
      {
        params: {
          part: "snippet,replies",
          videoId: videoId,
          key: APIKey,
          maxResults: 10,
          order: "relevance", // 인기순으로 댓글 가져오기
        },
      }
    );

    // const video = response.data.items[0]; // 첫 번째 결과
    const video = response.data; // 첫 번째 결과
    if (video) {
      console.log(video);
      const filePath = path.join(__dirname, "video.json");
      fs.writeFileSync(filePath, JSON.stringify(video, null, 2));
    } else {
      console.log("❌ 해당 비디오를 찾을 수 없습니다.");
    }
  } catch (error) {
    console.error(
      "오류 발생:",
      error.response ? error.response.data : error.message
    );
  }
}

async function getChannelId(channelName) {
  try {
    const response = await axios.get(
      "https://www.googleapis.com/youtube/v3/search",
      {
        params: {
          part: "snippet",
          q: channelName,
          type: "channel",
          key: APIKey,
        },
      }
    );

    if (response.data.items.length > 0) {
      const channel = response.data.items[0].id;
      return channel.channelId;
    } else {
      console.log("No channel found with the specified username.");
      return null;
    }
  } catch (error) {
    console.error(
      "Error fetching channel ID:",
      error.response?.data || error.message
    );
    return null;
  }
}

async function getVideoIdsFromChannel(channelId) {
  const YOUTUBE_API_URL = "https://www.googleapis.com/youtube/v3/";
  const videoIds = [];
  let nextPageToken = "";
  try {
    do {
      const response = await axios.get(`${YOUTUBE_API_URL}search`, {
        params: {
          key: APIKey,
          channelId,
          part: "id",
          maxResults: 50,
          type: "sh",
          pageToken: nextPageToken,
        },
      });

      const items = response.data.items;
      items.forEach((item) => {
        videoIds.push(item.id.videoId);
      });

      nextPageToken = response.data.nextPageToken || "";
    } while (nextPageToken);

    console.log(`Total videos fetched: ${videoIds.length}`);
    return videoIds;
  } catch (error) {
    console.error(
      "Error fetching video IDs:",
      error.response?.data || error.message
    );
    return [];
  }
}
```
