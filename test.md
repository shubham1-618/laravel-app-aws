I'll provide a comprehensive set of test commands to demonstrate the backend API functionality:

````bash
# 1. Test Registration
curl -X POST http://3.6.86.139/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "username": "testuser1",
      "email": "test1@test.com",
      "password": "password123"
    }
  }'

# 2. Test Login
curl -X POST http://3.6.86.139/api/users/login \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "test1@test.com",
      "password": "password123"
    }
  }'

# 3. Get Current User (use token from login response)
curl -X GET http://3.6.86.139/api/user \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json"

# 4. Create Article
curl -X POST http://3.6.86.139/api/articles \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "article": {
      "title": "Test Article",
      "description": "This is a test article",
      "body": "Article body goes here",
      "tagList": ["test", "demo"]
    }
  }'

# 5. Get Articles List
curl -X GET "http://3.6.86.139/api/articles?limit=10&offset=0" \
  -H "Content-Type: application/json"

# 6. Get Single Article
curl -X GET http://3.6.86.139/api/articles/test-article \
  -H "Content-Type: application/json"

# 7. Add Comment to Article
curl -X POST http://3.6.86.139/api/articles/test-article/comments \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "This is a test comment"
    }
  }'

# 8. Get Article Comments
curl -X GET http://3.6.86.139/api/articles/test-article/comments \
  -H "Content-Type: application/json"

# 9. Favorite Article
curl -X POST http://3.6.86.139/api/articles/test-article/favorite \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json"

# 10. Get User Profile
curl -X GET http://3.6.86.139/api/profiles/testuser1 \
  -H "Content-Type: application/json"

# 11. Follow User
curl -X POST http://3.6.86.139/api/profiles/testuser1/follow \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json"

# 12. Get Tags
curl -X GET http://3.6.86.139/api/tags \
  -H "Content-Type: application/json"

# 13. Update Article
curl -X PUT http://3.6.86.139/api/articles/test-article \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "article": {
      "title": "Updated Test Article",
      "description": "Updated description",
      "body": "Updated body"
    }
  }'

# 14. Delete Article
curl -X DELETE http://3.6.86.139/api/articles/test-article \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json"

# 15. Update User
curl -X PUT http://3.6.86.139/api/user \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newemail@test.com",
      "bio": "This is my new bio",
      "image": "https://i.imgur.com/example.jpg"
    }
  }'
````

To use these commands effectively:

1. Run them in sequence (registration first, then login, etc.)
2. Replace `YOUR_TOKEN_HERE` with the actual JWT token you receive from login
3. Replace `test-article` with the actual slug of your created article
4. Watch for the response codes and messages

Common HTTP Response Codes:
- 200: Success
- 201: Created
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 422: Validation Error

Would you like me to explain any specific command in more detail?
