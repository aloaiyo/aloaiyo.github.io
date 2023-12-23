---
layout: minimal
title: QuerySet을 count()로 row를 세는 것과 len()으로 세는 것의 차이
grand_parent: programming
parent: django
nav_order: 1
slug: queryset을-count-로-row를-세는-것과-len-으로-세는-것의-차이
date: 2023-12-21 00:00:00
last_modified_at: 2023-12-21 00:00:00
keywords:
    - queryset
---

# QuerySet을 count()로 row를 세는 것과 len()으로 세는 것의 차이

최근 받은 코드리뷰 중에 `len()`으로 row의 개수를 얻어오던 것을 `count`로 변경해달라는 코멘트를 받았다. stackoverflow를 몇 개 뒤져보니 row의 개수를 확인하는 것은 왠만하면 `count`를 이용해서 얻어오는 것을 다들 권장한다. 그렇다면 그 이유가 무엇인지 확인해보자.

현재 내 서비스의 User의 수를 한 번 알아본다고 하자.

```python
qs = User.objects.all()

# queryset의 count()를 사용하는 방법이 있다.
def user_count_by_count():
    return qs.count()

# len으로 확인 하는 방법과
def user_count_by_len():
    return len(qa)
```

이 때, `count()`의 경우 `SELECT COUNT(*) FROM User` 가 RDBS에서 실행되고 count가 python으로 전달될 것이다.

`len()`의 경우 `SELECT * FROM User`를 한 번 RDBS에서 실행하고 python으로 전달된 결과를 len()으로 셀 것이다.

## 핵심 내용
흔히들 row 개수를 셀 일이 있다면 count()를 쓰라고 하는데 그 이유는 `len()으로 count를 대신하려하면 fetch를 하면서 시간이 O(N)이 걸리고 이 결과를 웹서버의 memory에 담으면서 storage에 O(N)이 발생하며 복사하는 시간이 걸려 time에도 O(N)이 추가로 발생한다. 거기에 추가적으로 len()이 돌아가는 시간도 발생한다.` 이 두 과정은 인프라에 따라 다르겠지만 count가 2배정도 빠르다고 한다.

하지만 len()을 쓰는 것이 count보다 유리할 때가 있다.

```python
qs = User.object.all()

def prefer_len_to_count():
# User 정보 안에 telephone 필드가 있다고 가정하고 그 정보를 리스트로 취합한다.
    telephones = [p for p.telephone in qs]
       # 그 정보로 뭔가 작업을 하고...
    ...
    # qs의 개수를 얻는다 (용도는 알아서 생각해보자)
    len(qs)
```

위와 같이 queryset을 fetch해서 사용을 하는 상황이라면 len이 아주 쪼금 유리하다. 이유는 `count()`의 경우 RDBS에 두 번 (fetch 한 번, count() 한 번) 접근해야하기 때문이다.

엄청나게 많은(이 기준은 인프라 상황에 따라 너무 다르다) row를 셀 것이 아니라면 그냥 count써도 무방하다.

당연히 template에서도 count의 사용이 권장된다.

아래 count가

```javascript
{% raw %}{{ some_queryset.count }}{% endraw %}
```

아래의 len보다 더 권장된다.

```javascript
{% raw %}{{ len(some_queryset) }}{% endraw %}
```

참고 : [stackoverflow.com/questions/14327036/count-vs-len-on-a-django-queryset](stackoverflow.com/questions/14327036/count-vs-len-on-a-django-queryset)