# think4web

Для відключення капіталізації заголовків статей у `_layouts/post.html` змінив {{ page.title }} на {{ page.title | downcase | capitalize }} і у index.html змінив {{ post.title }} на {{ post.title | downcase | capitalize }}