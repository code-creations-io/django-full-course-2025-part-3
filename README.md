# django-full-course-2025-part-3

**Mastering Django Serializers – Building Production-Ready APIs (Courses App)**  
*Part 3 of the Django Full Course (2025 Edition)*

- Created at: 2025-10-23  
- Created by: `🐢 Arun Godwin Patel @ Code Creations`  

This part extends [Part 2](https://github.com/code-creations-io/django-full-course-part-2), where we defined our models for courses, modules, lessons, and users.  
Now, we’ll expose these models through **Django REST Framework (DRF) serializers** — enabling a robust and production-ready API layer.

---

## 🧩 Overview

In this part, you’ll learn how to:
- Serialize models with relationships (`ForeignKey`, `ManyToMany`, `OneToOne`)
- Handle nested and dynamic serializers
- Implement computed fields and custom validation
- Optimize API performance with `select_related` and `prefetch_related`
- Structure serializers for scalability and clean separation of logic

---

## ⚙️ Setup

Ensure DRF is installed and added to your project:
```bash
pip install djangorestframework
```

In `settings.py`:
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'core',
    'users',
]
```

---

## 🧱 Serializer Overview

| Serializer | Description |
|-------------|--------------|
| `UserProfileSerializer` | Serializes the `UserProfile` model, including linked `User`. |
| `LessonSerializer` | Handles `Lesson` fields and computed durations. |
| `ModuleSerializer` | Includes nested lessons. |
| `CourseSerializer` | Serializes relationships with `tags`, `topics`, and instructors, with computed completion. |
| `EnrollmentSerializer` | Handles course-user enrollment with computed progress. |
| `LessonProgressSerializer` | Tracks completion of lessons per user. |

---

## 👤 `users/serializers.py`

```python
from rest_framework import serializers
from django.contrib.auth.models import User
from users.models import UserProfile

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 'last_name']

class UserProfileSerializer(serializers.ModelSerializer):
    user = UserSerializer(read_only=True)

    class Meta:
        model = UserProfile
        fields = ['id', 'user', 'display_name', 'bio']
```

---

## 📘 `core/serializers.py`

```python
from rest_framework import serializers
from core.models import Course, Module, Lesson, Enrollment, LessonProgress
from users.serializers import UserSerializer

# ------------------ Lesson ------------------
class LessonSerializer(serializers.ModelSerializer):
    class Meta:
        model = Lesson
        fields = ['id', 'name', 'slug', 'content', 'duration_seconds', 'order', 'created_at', 'updated_at']

# ------------------ Module ------------------
class ModuleSerializer(serializers.ModelSerializer):
    lessons = LessonSerializer(many=True, read_only=True)

    class Meta:
        model = Module
        fields = ['id', 'name', 'slug', 'description', 'order', 'lessons', 'created_at', 'updated_at']

# ------------------ Course ------------------
class CourseSerializer(serializers.ModelSerializer):
    modules = ModuleSerializer(many=True, read_only=True)
    instructors = UserSerializer(many=True, read_only=True)
    total_lessons = serializers.SerializerMethodField()
    completion_rate = serializers.SerializerMethodField()

    class Meta:
        model = Course
        fields = [
            'id',
            'name',
            'slug',
            'description',
            'is_published',
            'tags',
            'topics',
            'instructors',
            'modules',
            'total_lessons',
            'completion_rate',
            'created_at',
            'updated_at',
        ]

    def get_total_lessons(self, obj):
        return obj.total_lessons()

    def get_completion_rate(self, obj):
        user = self.context.get('request').user if self.context.get('request') else None
        if user and user.is_authenticated:
            return obj.completion_for(user)
        return None

# ------------------ Enrollment ------------------
class EnrollmentSerializer(serializers.ModelSerializer):
    course = CourseSerializer(read_only=True)
    user = UserSerializer(read_only=True)
    progress_percent = serializers.SerializerMethodField()

    class Meta:
        model = Enrollment
        fields = ['id', 'course', 'user', 'enrolled_at', 'progress_percent']

    def get_progress_percent(self, obj):
        return obj.progress_percent()

# ------------------ LessonProgress ------------------
class LessonProgressSerializer(serializers.ModelSerializer):
    user = UserSerializer(read_only=True)
    lesson = LessonSerializer(read_only=True)

    class Meta:
        model = LessonProgress
        fields = ['id', 'user', 'lesson', 'completed', 'completed_at', 'created_at', 'updated_at']
```

---

## ⚡ Advanced Concepts

### ✅ Custom Validation Example

```python
class LessonSerializer(serializers.ModelSerializer):
    class Meta:
        model = Lesson
        fields = '__all__'

    def validate_duration_seconds(self, value):
        if value > 3600 * 4:
            raise serializers.ValidationError("Lesson duration cannot exceed 4 hours.")
        return value
```

---

### 🧩 Dynamic Fields (Context Aware)

```python
class CourseMinimalSerializer(serializers.ModelSerializer):
    class Meta:
        model = Course
        fields = ['id', 'name', 'slug']

class CourseDynamicSerializer(serializers.ModelSerializer):
    class Meta:
        model = Course
        fields = ['id', 'name', 'slug', 'description', 'modules']

    def __init__(self, *args, **kwargs):
        fields = kwargs.pop('fields', None)
        super().__init__(*args, **kwargs)
        if fields:
            allowed = set(fields)
            existing = set(self.fields)
            for field in existing - allowed:
                self.fields.pop(field)
```

---

### 🚀 Optimizing Querysets in ViewSets

```python
class CourseViewSet(viewsets.ModelViewSet):
    queryset = Course.objects.select_related().prefetch_related('modules__lessons', 'instructors')
    serializer_class = CourseSerializer
```

---

### 🧠 Serializer Best Practices for Production

- ✅ Always define `Meta.fields` explicitly — avoid `__all__` in public APIs.  
- ⚙️ Use nested serializers for read-only relationships.  
- 🚀 Optimize queries using `select_related` and `prefetch_related`.  
- 🔒 Perform validation and permission checks in serializers for safer writes.  
- 💾 Use pagination and filtering for large datasets.  
- 🧰 Cache frequently accessed serializers when appropriate.  

---

## 🧭 Next Steps

In **Part 4**, we’ll connect these serializers to **ViewSets and Routers**, building RESTful endpoints for courses, lessons, and user progress.

---

**📘 Resources**
- [Django REST Framework Docs](https://www.django-rest-framework.org/)
- [Advanced DRF Performance](https://testdriven.io/blog/drf-performance/)
- [Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/en/latest/)

---

**🧑‍💻 Created by**: `🐢 Arun Godwin Patel`  
**Brand**: `Code Creations`  
**Year**: 2025
