# django-full-course-2025-part-3

**Mastering Django Serializers – Building Production-Ready APIs**  
*Part 3 of the Django Full Course (2025 Edition)*

- Created at: 2025-10-23  
- Created by: `🐢 Arun Godwin Patel @ Code Creations`  

This module builds upon [Part 2](https://github.com/code-creations-io/django-full-course-part-2), where we structured our Django models and connected them to the database.  
In **Part 3**, we take the next big step — transforming our models into **JSON-ready API representations** using Django REST Framework (DRF) serializers.

---

## 🧩 Overview

Serializers are the **bridge between Django models and JSON data**.  
In production-ready APIs, serializers do much more than convert data — they handle:
- **Validation** (custom rules, cross-field checks)
- **Nested relationships** (ForeignKeys, ManyToMany)
- **Optimized queries** using `select_related` and `prefetch_related`
- **Dynamic fields and context-aware logic**
- **Integration with permissions, throttling, and pagination**

By the end of this part, you’ll be able to build **scalable, secure, and efficient REST APIs** that can power real-world frontends, mobile apps, and AI systems.

---

## ⚙️ Prerequisites

Ensure you’ve completed Part 2 and have the following setup:
```bash
python -m venv venv
source venv/bin/activate     # or venv\Scripts\activate on Windows
pip install -r requirements.txt
python manage.py runserver
```

---

## 🧠 Topics Covered

| # | Concept | Description |
|---|----------|-------------|
| 1 | **Serializer Basics** | Understanding `ModelSerializer` vs `Serializer`, how fields are auto-generated, and when to override them. |
| 2 | **Custom Validation** | Implementing field-level, object-level, and cross-field validation methods. |
| 3 | **Nested Serializers** | Serializing relationships (e.g. `Author` → `Post`, `Post` → `Comment`). |
| 4 | **SerializerMethodField** | Adding computed fields (e.g. `comment_count`, `is_popular`). |
| 5 | **Dynamic Fields** | Conditionally including/excluding fields using request context. |
| 6 | **Optimized Querysets** | Combining serializers with `select_related`, `prefetch_related` to reduce N+1 queries. |
| 7 | **Hyperlinked & Slug Serializers** | Linking objects via URLs or slugs instead of IDs. |
| 8 | **Nested Writes** | Handling creation/updating of nested models safely. |
| 9 | **Permissions & Validation Integration** | Combining serializers with DRF permissions and custom business logic. |
| 10 | **Performance & Caching** | Using `Serializer.cache` and DRF extensions for high-performance APIs. |

---

## 🧩 Example: Basic ModelSerializer

Let’s revisit the `Post` model from Part 2 and create a simple serializer.

```python
from rest_framework import serializers
from blog.models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['id', 'title', 'summary', 'author', 'created_at']
```

---

## 🧠 Example: Nested Serializers

The `Post` model relates to `Comment` through a ForeignKey.  
We can serialize both models together:

```python
from rest_framework import serializers
from blog.models import Post, Comment

class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ['id', 'author', 'content', 'created_at']

class PostSerializer(serializers.ModelSerializer):
    comments = CommentSerializer(many=True, read_only=True)

    class Meta:
        model = Post
        fields = ['id', 'title', 'summary', 'body', 'author', 'comments']
```

---

## 💡 SerializerMethodField for Computed Values

```python
class PostSerializer(serializers.ModelSerializer):
    comment_count = serializers.SerializerMethodField()

    class Meta:
        model = Post
        fields = ['id', 'title', 'summary', 'comment_count']

    def get_comment_count(self, obj):
        return obj.comments.count()
```

---

## 🧱 Advanced: Dynamic Fields & Context

```python
class DynamicPostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['id', 'title', 'summary', 'body', 'author']

    def __init__(self, *args, **kwargs):
        fields = kwargs.pop('fields', None)
        super().__init__(*args, **kwargs)
        if fields is not None:
            allowed = set(fields)
            for field_name in set(self.fields) - allowed:
                self.fields.pop(field_name)
```

You can now use it dynamically in views:
```python
serializer = DynamicPostSerializer(post, fields=['id', 'title'])
```

---

## 🔐 Validations & Permissions

```python
class SecurePostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['title', 'summary', 'body']

    def validate_title(self, value):
        if "spam" in value.lower():
            raise serializers.ValidationError("Title contains banned word: spam")
        return value

    def validate(self, data):
        if data['title'] == data['summary']:
            raise serializers.ValidationError("Title and summary cannot be identical")
        return data
```

---

## 🚀 Optimizing Querysets in Views

Avoid N+1 queries when serializing relationships:

```python
class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.select_related('author').prefetch_related('comments')
    serializer_class = PostSerializer
```

---

## 🧠 Bonus: Serializer Performance Tips

- Use `select_related` and `prefetch_related`
- Avoid unnecessary nested serialization for list endpoints
- Use `SerializerMethodField` sparingly
- For heavy APIs, consider DRF-extensions caching
- Benchmark using Django Debug Toolbar

---

## 🧰 Challenge Exercise

Create a serializer for the `Author` model that:
1. Displays all related posts with only `title` and `summary`.
2. Adds a computed field `total_posts`.
3. Enforces a validation rule that name cannot contain numbers.

---

## 🧭 Next Steps

In **Part 4**, we’ll explore **ViewSets, Routers, and Advanced API Design**, connecting our serializers to powerful REST endpoints.

---

**📘 Resources**
- [Django REST Framework Docs](https://www.django-rest-framework.org/)
- [Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/en/latest/)
- [DRF Performance Tips](https://testdriven.io/blog/drf-performance/)

---

**🧑‍💻 Created by**: `🐢 Arun Godwin Patel`  
**Brand**: `Code Creations`  
**Year**: 2025
