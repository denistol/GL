# Schema examples

## Session schema

```
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "ip": "123.123.123.1",
  "userAgent": "Chrome",
  "refreshTokenHash": "...",
  "expiresAt": "...",
  "createdAt": "..."
  deviceType: "MOBILE | DESKTOP | OTHER",
  fingerprint: "...",
}
```

## User schema

```
{
  "_id": "ObjectId",
  "email": "user@example.com",
  "name": "John Doe",
  "passwordHash": "...",
  "createdAt": "...",
  "lastLoginAt": "...",
  "isBlocked": false,
  "country": "US",
  "roles": ["user"], // Not shure about roles, Because all visitors have same permissions (as simple visitor). Maybe for future, we can define Role schema
  "preferences": {
      "language": "en",
      "theme": "dark"
  }
}
```

## Achievement schema

```
{
  "_id": "ObjectId",
  "title": "...",
  "description": "...",
  "media": ["media_id_1", "media_id_2"],
}

```

## User progress

```
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "gameId": "StaticContentId",
  "level": 10,
  "points": 1500,
  "achievements": ["ach_id_1", "ach_id_2"],
  "lastActivityAt": "...",
  "personalizedOffers": ["CampaignId1", "CampaignId2"]
}

```

## Static content schema

```
{
  "_id": "ObjectId",
  "type": "GAME|NEWS|ANNOUNCEMENT|FAQ",
  "title": "Game Title / News Headline",
  "slug": "game-title",
  "media": ["media_id_1", "media_id_2"],
  "tags": ["action", "multiplayer"], // Or crate schema like ContentTag
  "createdAt": "...",
  "updatedAt": "..."
  "content": "Markdown / HTML / JSON",
}
```

## Media assets schema

```
{
  "_id": "ObjectId",
  "fileName": "screenshot.png",
  "url": "https://s3.amazonaws.com/...",
  "mimeType": "image/png",
  "sizeBytes": 123456,
  "uploadedBy": "userId",
  "relatedContentId": "StaticContentId",
  "createdAt": "..."
}
```

## AB Rule schema
```
  "_id": "ObjectId",
  "name": "AB exampl",
  "description": "AB example",
  "startAt": "...",
  "endAt": "...",
  "active": true,
  "rule": {
    // Here we can describe rule for A|B variant
  }

```

## Marketing campaigns schema

```
{
  "_id": "ObjectId",
  "name": "Spring Promo",
  "description": "Spring marketing campaign",
  "startAt": "...",
  "endAt": "...",
  "active": true,
  "targetingRules": { // Or we can create other schema for store mappings IP -> COUNTRY
      "countries": ["US", "CA"],
      "ipRanges": ["192.168.0.0/16", "10.0.0.0/8"],
      "deviceTypes": ["MOBILE", "DESKTOP"]
  },
  "contentIds": ["StaticContentId1", "StaticContentId2"],

  "abVariants": [
    "ab_rule_id_1", "ab_rule_id_2", // ... We can add more than A, B rule, i think its more flexible.
  ],
  "createdAt": "...",
  "updatedAt": "..."
}
```

## AB Experiment Log

```
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "campaignId": "ObjectId",
  "variantId": "ab_rule_variant_id",
  "assignedAt": "...",
  "conversionEvent": timestamp // Conversion event timestapm
}
```

## Notification/Message schema

```
{
  "_id": "ObjectId",
  "type": "EMAIL | PUSH | IN_APP | MEDIA_PROCESSING",
  "recipientUserId": "ObjectId",        // null for broadcast messages
  "payload": {
    "subject": "Your level up!",
    "body": "Congratulations! You reached level 10.",
    "mediaId": "media_id_1"            // optional, for media processing or attachments
  },
  "status": "PENDING | SENT | FAILED",
  "retryCount": 0,
  "scheduledAt": "...",                  // optional, delayed delivery
  "createdAt": "...",
  "sentAt": "..."                        // timestamp when message was successfully processed
}
```