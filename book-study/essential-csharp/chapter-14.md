# 14장 — 이벤트

## event 키워드

대리자에는 여러 메서드를 참조하거나 참조 해제할 수 있다.  
이 때문에 여러 클래스에서 하나의 이벤트에 접근 가능한 경우 다음과 같은 문제가 발생할 수 있다.

- 이미 메서드가 할당되어 있는 대리자를 초기화해버리거나
- 덮어서 할당하는 경우

`event` 키워드는 해당 대리자를 외부 클래스에서 호출할 수 없게 하고, 오직 구독/구독 취소(`+=`, `-=`)만 가능하도록 제한한다.

```csharp
internal class Program
{
    public event Action ActionEvent = null;

    public void Method()
    {
        // 클래스 내부에서는 직접 할당 가능
        ActionEvent = () => { bool? a = false; };
    }
}

public class OutClass
{
    Program program = new Program();

    public void Method()
    {
        // 외부 할당 불가능
        // program.ActionEvent = null;  // 오류

        // 외부 호출 불가능
        // program.ActionEvent.Invoke();  // 오류

        // 구독/구독 취소만 가능
        program.ActionEvent += () => { };
        program.ActionEvent -= () => { };
    }
}
```

---

## add / remove 접근자

프로퍼티와 유사한 형태로 구독/구독 취소를 커스터마이징할 수 있다.

```csharp
protected Action? ActionEvent = null;

public event Action ValueChange
{
    add
    {
        ActionEvent = (Action)System.Delegate.Combine(value, ActionEvent);
    }

    remove
    {
        ActionEvent = (Action?)System.Delegate.Remove(ActionEvent, value);
    }
}
```

---

[← 목차](./README.md)
