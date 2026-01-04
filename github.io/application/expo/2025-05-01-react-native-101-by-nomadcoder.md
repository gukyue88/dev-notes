---
layout: single
title: "[expo] React Native 101 노마드코더 내용 요약"
categories: application/expo
---

### React Native란

- bridge를 통해 React native 코드를 안드로이드, IOS 등으로 UI등을 요청하는거임, 그래서 안드로이드에서 만드는 버튼과 IOS에서 만드는 버튼의 모양이 다른거임

![[스크린샷 2023-06-04 오후 10.50.33.png]]

![[스크린샷 2023-06-04 오후 10.52.08.png]]

### expo cli 설치

```sh
npm install --global expo-cli
```

### 프로젝트 생성

```sh
# expo init <app-name> # legacy command
#WARNING: The legacy expo-cli does not support Node +17. Migrate to the new local Expo CLI: https://blog.expo.dev/the-new-expo-cli-f4250d8e3421.

#Migrate to using:
# npx create-expo-app --template
# Navigation (Typescript) 선택시 npm run start가 작동하지 않음...
npx create-expo-app@latest --example with-router

cd <app-name>
npm start
```

### expo login

```sh
expo login
```

### Snack

snack.expo.dev 홈페이지에 들어가면 기기 없이도 빌드가 가능함

### React Native의 규칙

- React Native는 웹사이트가 아니기 때문에 div 같은걸 사용할 수 없음. 대신 `<View>`를 사용함
- React Native에 있는 모든 텍스트는 `<Text>` 컴포넌트 안에 들어있어야 함
- 웹에서 사용하던 style과 거의 유사하나 동작하는 방법이 다르거나 못쓰는 것들도 존재함
- StyleSheet.create를 사용하면 자동완성 기능을 활용할 수 있기 때문에 사용을 권장함
- StatusBar는 컴포넌트이지만 보이지 않음. OS의 상태바를 제어하는 기능을 가짐

### Layout System

- View는 기본적으로 FlexBox임
- 기본적으로 Direction이 세로임 (모바일)
- width, height를 사용하지 않고, flex에 사이즈를 적어서 여러 화면에 대응해야함

### ScrollView

스크롤 할 수 있는 뷰 컴포넌트

```js
<ScrollView
  horizontal
  contentContainerStyle={styles.contentContainer}
  pagingEnabled
  showHorizontalScrollIndicator={false}
></ScrollView>
```

- horizontal : 횡 스크롤
- contentContainerStyle: style 대신 contentContainerStyle로 스타일 변경
- pagingEnabled : 부드럽게 스크롤 하는대신 아이템을 하나씩 페이징시켜서 끊어 보여줌
- showHorizontalScrollIndicator : 횡방향 스크롤 인디케이터 숨길 때 false

### Dimensions

화면 크기 얻기

```js
const { height: SCREEN_HEIGHT, width: SCREEN_WIDTH } = Dimensions.get("window");
```

### expo-location

유저의 위치 확인

```sh
expo install expo-location # 설치
```

```js
import * as Location from "expo-location";

const [city, setCity] = useState("Loading...");
const [ok, setOk] = useState(true);
const ask = async () => {
  const { granted } = await Location.requestForegroundPermissionsAsync();
  if (!granted) {
    setOk(false);
  }
  const {
    coords: { latitude, longitude },
  } = await Location.getCurrentPositionAsync({ accuracy: 5 });
  const location = await Location.reverseGeocodeAsync(
    { latitude, longitude },
    { useGoogleMaps: false }
  );
  setCity(location[0].city);
};

useEffect(() => {
  getWeather();
}, []);
```

### ActivityIndicator

로딩중 표시

### expo/vector-icons

아이콘
icons.expo.fyi

```jsx
import { Fontisto } from "@expo/vector-icons";

<Fontisto name="cloudy" size={24} color="white" />;
```

- family 이름과 그 안에 상세 아이콘 이름을 사용

### TouchableOpacity

투명 애니메이션 효과가 적용된 터치이벤트 적용 가능한 컴포넌트

- 적용할 요소를 TouchableOpacity로 감쌈

```jsx
<TochableOpacity onPress={onClick}>
  <Text>Click</Text>
</TochableOpacity>
```

### TextInput

react native 에서 유저에게 텍스트를 입력 받는 유일한 방법

```jsx
const [text, setText] = useState("");
const onChangeText = (payload) => setText(payload);

<TextInput
  placeholder="input something..."
  onChangeText={onChangeText}
  keyboardType="numeric"
  returnKeyType="send"
  value={text}
  onSubmitEditing={onSubmit}
/>;
```

### AsyncStorage

```sh
expo install ...
```

```js
import AsyncStorage from "@react..."

const storeData = async (value) = {
	try {
		await AsyncStorage.setItem('@storage_Key', value);
	} catch (e) {
		// saving error
	}
}

const getData = async () => {
	try {
		const jsonValue = await AsyncStorage.getItem("@storage_Key");
		return jsonValue != null ? JSON.parse(jsonValue) : null;
	} catch(e) {

	}
}
```

### SaveAreaView

iphone 위에 패딩 자동으로 들어가는 뷰
