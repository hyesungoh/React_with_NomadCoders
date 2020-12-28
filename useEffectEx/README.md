## React Hooks?

state와 life cycle을 확인하기 위해 class component를 사용해 왔지만 this, setState, render 등 코드가 길고 커지는 것을 해소하기 위해 제작됨

덕분에 hook들을 이용하여 함수형 component를 사용하여 state와 life cycle 제어가 가능하게 됨

#### [useState 보러가기](https://github.com/hyesungoh/React_with_NomadCoders/tree/master/useStateEx)

---

#### Basic useEffect

```jsx
useEffect(function);
```

-   위와 같이 사용 시 componentDidMount, componentDidUpdate와 같이 호출됨

```jsx
useEffect(function, [someValue]);
```

-   두 번째 인자가 있을 시 해당 인자가 update될 때 호출 + componentDidMount

```jsx
useEffect(function, [];)
```

-   componentDidMount만 사용하고 싶을 시 빈 인자를 사용하면 됨

#### useTitle

-   html head의 title을 mount, update

```jsx
const useTitle = (initialTitle) => {
    const [title, setTitle] = useState(initialTitle);

    // html head의 title의 값을 바꿈
    const updateTitle = () => {
        const htmlTitle = document.querySelector("title");
        htmlTitle.innerText = title;
    };

    useEffect(updateTitle, [title]); // state인 title이 변경 시 updateTitle 함수 호출
    return setTitle;
};

const App = () => {
    const titleUpdater = useTitle("Loading..");
    // setTimeout(() => {
    //     titleUpdater("HOME");
    // }, 3000);

    return (
        <div className="App">
            <h1> hello </h1>
        </div>
    );
};
```

#### useRef

-   document.getElementById와 같이 사용가능

```jsx
const something = useRef();

<input ref={something} />;
```

#### useEffect > componentWillUnMount

-   useEffect에 fucntion을 사용 시 mount, update 때 호출 됨

-   해당 function의 반환 값이 componentWillUnMount 때 호출 됨

```jsx
useEffect(() => {
    console.log("Mount");
    return () => console.log("UnMount);
});
```

#### useClick

```jsx
const useClick = (onClick) => {
    // Click을 감지할 요소
    const checkItem = useRef();

    // componentDidMount시 EventListener를 부착
    useEffect(() => {
        if (checkItem.current) {
            checkItem.current.addEventListener("click", onClick);
        }

        // componentWillUnMount시 EventListener 탈착
        return () => {
            if (checkItem.current) {
                checkItem.current.removeEventListener("click", onClick);
            }
        };
    }, []);

    return checkItem;
};

const App = () => {
    // Click 시 호출될 함수
    const sayHello = () => console.log("hello");
    const title = useClick(sayHello);
    return (
        <div className="App">
            <h1 ref={title}> hello </h1>
        </div>
    );
};
```

#### useConfirm

-   window.confirm을 이용하여 간단한 함수형 useConfirm 작성

```jsx
const useConfirm = (message = "", onTrue, onFalse) => {
    // 함수가 존재하지 않거나 형식이 함수일 때
    if (!onTrue || typeof onTrue !== "function") {
        return;
    }
    if (!onFalse || typeof onFalse !== "function") {
        return;
    }

    const confirmAction = () => {
        if (window.confirm(message)) {
            onTrue();
        } else {
            onFalse();
        }
    };
    return confirmAction;
};

const App = () => {
    const deleteWorld = () => console.log("DELETE THE WORLD");
    const saveWorld = () => console.log("SAVE THE WORLD ONE MORE");
    const confirmer = useConfirm("진짜루..?", deleteWorld, saveWorld);

    return (
        <div className="App">
            <button onClick={confirmer}>DELETE WORLD</button>
        </div>
    );
};
```

#### usePreventLeave

-   `beforeunload` 를 이용하여 사용자가 웹을 나갈 때 확인 가능

```jsx
const usePreventLeave = () => {
    const listener = (event) => {
        event.preventDefault();
        event.returnValue = "";
    };
    const enablePrevent = () =>
        window.addEventListener("beforeunload", listener);
    const disablePrevent = () =>
        window.removeEventListener("beforeunload", listener);

    return { enablePrevent, disablePrevent };
};

const App = () => {
    const { enablePrevent, disablePrevent } = usePreventLeave();
    return (
        <div className="App">
            <button onClick={enablePrevent}>PROTECT</button>
            <button onClick={disablePrevent}>UNPROTECT</button>
        </div>
    );
};
```

#### useBeforeLeave

-   `mouseleave` 를 이용하여 사용자의 마우스가 document 영역 밖으로 나갈 시를 확인

```jsx
const useBeforeLeave = (action) => {
    const handle = (event) => {
        const { clientY } = event;
        if (clientY <= 0) {
            action();
        }
    };

    useEffect(() => {
        document.addEventListener("mouseleave", handle);

        return () => document.removeEventListener("mouseleave", handle);
    }, []);
    // useEffect가 호출안되는 경우 존재시 에러 발생
    if (typeof action !== "function") {
        return;
    }
};

const App = () => {
    const beforeLeaveAction = () => {
        console.log("leaving");
    };
    useBeforeLeave(beforeLeaveAction);

    return (
        <div className="App">
            <h1>hello</h1>
        </div>
    );
};
```