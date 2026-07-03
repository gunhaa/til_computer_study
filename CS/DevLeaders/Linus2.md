# The Mind Behind Linux | Linus Torvalds | TED

> https://www.youtube.com/watch?v=o8NPllzkFhE

- Linus는 자신이 즐길 수 있는 일을 업으로 삼아서, 지금의 linux를 만들었다
- 자신의 강점과 약점을 정확히 알아서 현재의 linus가 있을 수 있던 것 같다
  - 협업이 힘든 것을 오픈 소스 작업으로 해결했다
  - 이메일을 버퍼처럼 사용해 사람과의 소통이 약점인 것을 커버했다
  - 여러가지 의미로 오픈소스에 최적화 된 사람이다
  - 자기 겍관화가 정말 잘 되있는 사람이다

## linus good taste code

```C
struct list_item {
        int value;
        struct list_item *next;
};
typedef struct list_item list_item;

struct list {
        struct list_item *head;
};
typedef struct list list;

// 흔히 보는 링크드리스트이며, linus가 좋은 코드는 아니라고 생각하는 코드
void remove_cs101(list *l, list_item *target)
{
        list_item *cur = l->head, *prev = NULL;
        while (cur != target) {
                prev = cur;
                cur = cur->next;
        }
        if (prev)
                prev->next = cur->next;
        else
                l->head = cur->next;
}

// 논리적인 코드로 필요 없는 if문을 제거한 더 우아한 방법
void remove_elegant(list *l, list_item *target)
{
        list_item **p = &l->head;
        while (*p != target)
                p = &(*p)->next;
        *p = target->next;
}
```

- linus는 커널개발자라 if문을 싫어 한다(cpu 분기 예측의 실패로 인한 오버헤드)
- 일반적으로 if문은 큰 생각 없이 올바른 논리로 연결에 실패한 코드일 가능성이 높다
  - 코드를 작성할 떄 본질을 꿰뚫어 보고 핵심을 관통하는 하나의 규칙을 찾아내는 능력이 필요하다
    - 본질을 짚어낸다면 if문이 덜 한 코드를 작성할 수 있을 가능성이 높다
  - if문이 진짜 필요한 경우도 있으나, 한 번쯤은 생각해보라는 의미로 한 말인 것 같다
  - 언제나 더 좋은 코드로 읽기 쉽게 만들라는 의미로 받아들이는 것이 좋을 것 같다