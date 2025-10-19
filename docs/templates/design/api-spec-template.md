# UC-XXX: [機能名] - API仕様

## エンドポイント
- **メソッド**: POST/GET/PUT/DELETE
- **パス**: `/api/v1/xxx`
- **認証**: 必要/不要

## リクエスト

### ヘッダー
```
Content-Type: application/json
Authorization: Bearer {token}  # 認証が必要な場合
```

### リクエストボディ
```json
{
  "field1": "value1",
  "field2": 123,
  "field3": {
    "nestedField": "value"
  }
}
```

### パラメータ説明
| パラメータ | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| field1 | string | ○ | フィールド1の説明 |
| field2 | number | ○ | フィールド2の説明 |
| field3.nestedField | string | × | ネストされたフィールドの説明 |

## レスポンス

### 成功レスポンス (200 OK)
```json
{
  "status": "success",
  "data": {
    "id": 123,
    "result": "processed"
  }
}
```

### レスポンス項目説明
| 項目 | 型 | 説明 |
|------|-----|------|
| status | string | 処理結果のステータス |
| data.id | number | 生成されたID |
| data.result | string | 処理結果 |

## エラーレスポンス

### 400 Bad Request (バリデーションエラー)
```json
{
  "status": "error",
  "message": "Validation failed",
  "errors": [
    {
      "field": "field1",
      "message": "必須項目です"
    }
  ]
}
```

### 401 Unauthorized (認証エラー)
```json
{
  "status": "error",
  "message": "Unauthorized"
}
```

### 500 Internal Server Error (サーバーエラー)
```json
{
  "status": "error",
  "message": "Internal server error"
}
```

## 備考
[その他の補足情報]
