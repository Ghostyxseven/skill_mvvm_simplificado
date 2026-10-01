# Recursos nativos (câmera, localização, etc.) no MVVM

O Model **nunca** importa `expo-*`. Então, ao usar um recurso nativo:

| Peça | Onde fica |
|---|---|
| Componente visual (`CameraView`, mapa) | **View** |
| Hook de permissão do SDK (`useCameraPermissions`) | **ViewModel** (hooks são permitidos) |
| API não visual (`expo-location`, `expo-sqlite`, `AsyncStorage`) | **`infra/`**, atrás de uma interface em `model/services/` |
| Regra ("só salva foto com localização ou marcada como sem") | **Model/UseCase** |

No Simplificado, ao precisar do primeiro recurso nativo/SDK crie `src/infra/` e a interface em `model/services/` — é o primeiro passo da evolução para o Sofisticado, e o Model continua puro.

## Exemplo: tirar foto + registrar localização

```typescript
// src/model/entities/MyPhoto.ts
export type MyPhoto = {
  uri: string;
  latitude: number | null;
  longitude: number | null;
  timestamp: number;
};
```

```typescript
// src/model/services/ILocationService.ts
export interface ILocationService {
  /** Retorna null se sem permissão ou sem sinal; nunca lança erro de SDK. */
  getCurrentPosition(): Promise<{ latitude: number; longitude: number } | null>;
}
```

```typescript
// src/infra/services/ExpoLocationService.ts
import * as Location from "expo-location";
import { ILocationService } from "@/model/services/ILocationService";

export class ExpoLocationService implements ILocationService {
  async getCurrentPosition() {
    try {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== "granted") return null;
      const { coords } = await Location.getCurrentPositionAsync({});
      return { latitude: coords.latitude, longitude: coords.longitude };
    } catch {
      return null; // GPS desligado etc.: o erro técnico não vaza
    }
  }
}
```

```typescript
// src/viewmodel/useCameraViewModel.ts
import { useCameraPermissions } from "expo-camera";
import { useState } from "react";
import { ExpoLocationService } from "@/infra/services/ExpoLocationService";
import { MyPhoto } from "@/model/entities/MyPhoto";

const locationService = new ExpoLocationService(); // no Sofisticado, injetado pela Factory

export function useCameraViewModel() {
  const [permission, requestPermission] = useCameraPermissions();
  const [photos, setPhotos] = useState<MyPhoto[]>([]);
  const [saving, setSaving] = useState(false);

  async function addPhoto(uri: string) {
    setSaving(true);
    try {
      const position = await locationService.getCurrentPosition();
      setPhotos((current) => [
        { uri, latitude: position?.latitude ?? null, longitude: position?.longitude ?? null, timestamp: Date.now() },
        ...current,
      ]);
    } finally {
      setSaving(false);
    }
  }

  return { permission, requestPermission, photos, saving, addPhoto };
}
```

```tsx
// src/app/camera.tsx (trecho)
import { CameraView } from "expo-camera";
import { useRef } from "react";
import { useCameraViewModel } from "@/viewmodel/useCameraViewModel";

const Camera = () => {
  const cameraRef = useRef<CameraView>(null);
  const { permission, requestPermission, photos, saving, addPhoto } = useCameraViewModel();

  if (!permission) return null;                       // ainda carregando a permissão
  if (!permission.granted) { /* mensagem + Pressable chamando requestPermission */ }

  async function capture() {                          // o ref do componente visual fica na View
    const photo = await cameraRef.current?.takePictureAsync();
    if (photo) await addPhoto(photo.uri);             // a decisão do que fazer com a foto é da ViewModel
  }
  // <CameraView ref={cameraRef} .../> + Pressable onPress={capture} + FlatList de photos
};
export default Camera;
```

## Boas práticas

- Permissões podem ser revogadas a qualquer momento: trate "negada" com mensagem e caminho para reabilitar.
- Mostre indicador enquanto obtém a localização (`saving`).
- Não mantenha muitas fotos em memória/estado: persista metadados (SQLite/AsyncStorage via repository na Infra) e gere miniaturas.
- Declare permissões/plugins no `app.json` conforme a documentação de cada módulo Expo.
