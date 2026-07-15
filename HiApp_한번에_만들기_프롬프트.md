# 걷기 GPS 기록 앱 — 한 번에 만들기 프롬프트

아래 내용을 Claude에게 **처음부터 통째로** 복사해서 붙여넣으면, 이번에 겪었던 시행착오(npm 버그, yarn 버그, SDK 버전 문제) 없이 한 번에 완성됩니다.

---

## 복사해서 쓸 프롬프트

```
윈도우 PC에서 Expo로 앱을 만들려고 해. 다음 조건을 꼭 지켜서 처음부터 끝까지 순서대로 안내해줘.

[목표]
- 내가 걸은 위치의 GPS 좌표를 저장하고 지도 위에 경로(선)로 표시하는 앱
- "추적 시작" / "추적 종료" 버튼으로 걷는 동안 좌표를 계속 기록

[반드시 지켜야 할 환경 조건 - 과거에 겪은 문제들 때문]
1. npm 버전을 반드시 11.x로 맞출 것 (npm 12는 `npm pack --dry-run`의 JSON 출력 형식이 바뀌어서
   Expo 프로젝트 생성 도구가 깨짐). 먼저 `npm install -g npm@11` 실행 후 진행.
2. 프로젝트 생성은 yarn이 아니라 npx로 할 것 (yarn create는 --template 옵션을
   제대로 전달하지 못하는 버그가 있었음).
3. 프로젝트는 반드시 SDK 54로 만들 것 (Expo Go 앱의 Play스토어 버전이 최신 SDK를
   아직 지원하지 않는 경우가 많으므로). 아래 명령을 정확히 써서 만들어줘:
   npx create-expo-app@latest [프로젝트명] --template blank-typescript@sdk-54
   (tabs 구조 대신 blank 템플릿을 써서 index.tsx 위치가 app/index.tsx로 단순하게 되도록 할 것)
4. 프로젝트 생성 후 package.json에서 "expo": "~54..." 로 되어 있는지 반드시 확인하고
   알려줘.

[기능 요구사항]
- expo-location으로 위치 권한 요청 + 현재 위치 표시
- react-native-maps로 지도 표시, 걸은 경로를 Polyline으로 그리기
- "추적 시작" 누르면 3초 또는 5m 이동마다 좌표 저장, "추적 종료"로 멈춤
- app.json에 위치 권한 설명 문구도 plugins에 추가해줘

[진행 방식]
- 각 단계마다 내가 터미널에 입력할 정확한 명령어를 코드블럭으로 줘
- 코드는 전체 파일 내용을 한 번에 줘서 복사+붙여넣기만 하면 되게 해줘
- 실행(npx expo start) 후 QR코드를 갤럭시 폰 Expo Go 앱으로 스캔하는 것까지 안내해줘
```

---

## 참고: 이번에 실제로 겪었던 문제들 (기록용)

| 순서 | 문제 | 원인 | 해결 |
|---|---|---|---|
| 1 | `npx create-expo-app` 실행 시 JSON 파싱 오류 | npm 12(Node 24 번들)가 `npm pack --dry-run --json` 출력 형식을 바꿈 | `npm install -g npm@11`로 다운그레이드 |
| 2 | `yarn create expo-app --template ...` 옵션이 무시됨 | yarn이 인자를 셸에 전달하는 과정에서 옵션이 깨짐 | yarn 대신 npx 사용 |
| 3 | 폰에서 "Project is incompatible with this version of Expo Go" | 프로젝트가 SDK 57로 만들어졌는데, Play스토어 Expo Go는 SDK 54까지만 지원 (SDK 57 전환기) | `npx create-expo-app@latest --template default@sdk-54`로 SDK 54 프로젝트 생성 |
| 4 | `app/index.tsx`를 못 찾음 | 템플릿마다 폴더 구조가 다름 (tabs 구조는 `app/(tabs)/index.tsx`) | `dir /s /b app`으로 실제 구조 확인 후 정확한 파일 위치 찾기 |

---

## 완성된 최종 코드 (참고용 백업)

### `app/index.tsx` (또는 `app/(tabs)/index.tsx`)

```tsx
import { useState, useEffect, useRef } from 'react';
import { StyleSheet, View, Text, TouchableOpacity } from 'react-native';
import MapView, { Marker, Polyline } from 'react-native-maps';
import * as Location from 'expo-location';

type LatLng = { latitude: number; longitude: number };

export default function Index() {
  const [errorMsg, setErrorMsg] = useState<string | null>(null);
  const [currentRegion, setCurrentRegion] = useState<any>(null);
  const [path, setPath] = useState<LatLng[]>([]);
  const [tracking, setTracking] = useState(false);
  const subscriptionRef = useRef<Location.LocationSubscription | null>(null);

  useEffect(() => {
    (async () => {
      let { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        setErrorMsg('위치 권한이 거부되었습니다.');
        return;
      }
      let loc = await Location.getCurrentPositionAsync({});
      setCurrentRegion({
        latitude: loc.coords.latitude,
        longitude: loc.coords.longitude,
        latitudeDelta: 0.005,
        longitudeDelta: 0.005,
      });
    })();
  }, []);

  const startTracking = async () => {
    setPath([]);
    setTracking(true);
    subscriptionRef.current = await Location.watchPositionAsync(
      {
        accuracy: Location.Accuracy.High,
        timeInterval: 3000,
        distanceInterval: 5,
      },
      (loc) => {
        const newPoint: LatLng = {
          latitude: loc.coords.latitude,
          longitude: loc.coords.longitude,
        };
        setPath((prev) => [...prev, newPoint]);
      }
    );
  };

  const stopTracking = () => {
    subscriptionRef.current?.remove();
    subscriptionRef.current = null;
    setTracking(false);
  };

  if (errorMsg) {
    return (
      <View style={styles.center}>
        <Text>{errorMsg}</Text>
      </View>
    );
  }

  if (!currentRegion) {
    return (
      <View style={styles.center}>
        <Text>현재 위치를 가져오는 중...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <MapView style={styles.map} initialRegion={currentRegion} showsUserLocation>
        {path.length > 1 && (
          <Polyline coordinates={path} strokeColor="#1D9E75" strokeWidth={4} />
        )}
        {path.map((point, index) => (
          <Marker key={index} coordinate={point} pinColor="orange" />
        ))}
      </MapView>

      <View style={styles.buttonRow}>
        {!tracking ? (
          <TouchableOpacity style={styles.button} onPress={startTracking}>
            <Text style={styles.buttonText}>추적 시작</Text>
          </TouchableOpacity>
        ) : (
          <TouchableOpacity style={[styles.button, styles.stopButton]} onPress={stopTracking}>
            <Text style={styles.buttonText}>추적 종료</Text>
          </TouchableOpacity>
        )}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  map: { flex: 1 },
  center: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  buttonRow: { position: 'absolute', bottom: 40, alignSelf: 'center' },
  button: {
    backgroundColor: '#1D9E75',
    paddingVertical: 14,
    paddingHorizontal: 30,
    borderRadius: 30,
  },
  stopButton: { backgroundColor: '#D85A30' },
  buttonText: { color: 'white', fontWeight: 'bold', fontSize: 16 },
});
```

### `app.json`에 추가할 부분 (plugins 배열 안)

```json
[
  "expo-location",
  {
    "locationAlwaysAndWhenInUsePermission": "걸은 경로를 지도에 표시하기 위해 위치 정보가 필요합니다."
  }
]
```

### 필요한 패키지 설치 명령

```
npx expo install expo-location react-native-maps
```
