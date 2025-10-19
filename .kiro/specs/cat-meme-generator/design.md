# Design Document

## Overview

El Cat Meme Generator es una aplicación web de página única (SPA) que permite a los usuarios subir imágenes de sus gatos y generar memes automáticamente usando IA. La aplicación utiliza una arquitectura frontend-backend separada, donde el frontend maneja la interfaz de usuario y el backend procesa las imágenes y se comunica con la API de Anthropic.

## Architecture

### High-Level Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │    Backend      │    │  Anthropic API  │
│   (React/Vue)   │◄──►│   (Node.js)     │◄──►│                 │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │
         │                       │
         ▼                       ▼
┌─────────────────┐    ┌─────────────────┐
│  Local Storage  │    │  File Storage   │
│  (Temp Images)  │    │  (Temp Files)   │
└─────────────────┘    └─────────────────┘
```

### Technology Stack

**Frontend:**
- React.js con TypeScript para type safety
- Canvas API para manipulación de imágenes
- CSS Modules o Styled Components para estilos
- Axios para comunicación HTTP

**Backend:**
- Node.js con Express.js
- Multer para manejo de archivos
- Sharp para procesamiento de imágenes
- Anthropic SDK para integración con la API

## Components and Interfaces

### Frontend Components

#### 1. App Component
- Componente principal que maneja el estado global
- Coordina la comunicación entre componentes

#### 2. ImageUploader Component
```typescript
interface ImageUploaderProps {
  onImageUpload: (file: File) => void;
  isLoading: boolean;
}
```
- Maneja la selección y validación de archivos
- Muestra vista previa de la imagen seleccionada
- Valida formato y tamaño de archivo

#### 3. MemeGenerator Component
```typescript
interface MemeGeneratorProps {
  image: File | null;
  onMemeGenerated: (memeData: MemeData) => void;
}

interface MemeData {
  imageUrl: string;
  text: string;
  timestamp: number;
}
```
- Envía solicitudes al backend para generar memes
- Maneja estados de carga y error
- Permite generar múltiples variaciones

#### 4. MemeDisplay Component
```typescript
interface MemeDisplayProps {
  memes: MemeData[];
  currentIndex: number;
  onIndexChange: (index: number) => void;
}
```
- Muestra el meme generado con navegación
- Renderiza texto superpuesto usando Canvas
- Proporciona controles de navegación entre variaciones

#### 5. MemeCanvas Component
```typescript
interface MemeCanvasProps {
  image: string;
  text: string;
  onCanvasReady: (canvas: HTMLCanvasElement) => void;
}
```
- Maneja la renderización del meme usando Canvas API
- Aplica estilos de texto (fuente, color, sombra)
- Posiciona el texto de manera óptima

#### 6. DownloadButton Component
```typescript
interface DownloadButtonProps {
  canvas: HTMLCanvasElement | null;
  filename?: string;
}
```
- Convierte el canvas a imagen descargable
- Maneja la descarga del archivo

### Backend API Endpoints

#### POST /api/generate-meme
```typescript
interface GenerateMemeRequest {
  image: File;
  variation?: number;
}

interface GenerateMemeResponse {
  success: boolean;
  text?: string;
  error?: string;
}
```
- Recibe imagen del usuario
- Genera prompt contextual para Anthropic
- Retorna texto del meme generado

#### POST /api/upload-image
```typescript
interface UploadImageResponse {
  success: boolean;
  imageId?: string;
  error?: string;
}
```
- Valida y almacena imagen temporalmente
- Retorna ID único para referenciar la imagen

### Anthropic Integration

#### Prompt Strategy
```typescript
interface MemePrompt {
  systemPrompt: string;
  userPrompt: string;
  maxTokens: number;
  temperature: number;
}
```

**System Prompt:**
"Eres un experto en crear memes de gatos divertidos e incoherentes. Genera textos cortos (máximo 2 líneas) que sean absurdos, graciosos y apropiados para redes sociales."

**User Prompt Template:**
"Genera un texto de meme divertido e incoherente para esta imagen de un gato. El texto debe ser en español, absurdo pero gracioso, y apropiado para redes sociales. Formato: solo el texto del meme, sin explicaciones."

## Data Models

### Frontend State Management
```typescript
interface AppState {
  currentImage: File | null;
  memes: MemeData[];
  currentMemeIndex: number;
  isGenerating: boolean;
  error: string | null;
}

interface MemeData {
  id: string;
  imageUrl: string;
  text: string;
  timestamp: number;
  canvas?: HTMLCanvasElement;
}
```

### Backend Data Models
```typescript
interface ImageMetadata {
  id: string;
  originalName: string;
  mimeType: string;
  size: number;
  uploadTime: Date;
  tempPath: string;
}

interface MemeRequest {
  imageId: string;
  variationNumber: number;
  userId?: string; // Para futuras mejoras
}
```

## Error Handling

### Frontend Error Handling
- **File Validation Errors:** Mostrar mensajes específicos para formatos no soportados o archivos muy grandes
- **Network Errors:** Retry automático con backoff exponencial
- **API Errors:** Mostrar mensajes user-friendly basados en códigos de error
- **Canvas Errors:** Fallback a imagen sin texto si falla la renderización

### Backend Error Handling
- **File Upload Errors:** Validación de tamaño y formato con mensajes descriptivos
- **Anthropic API Errors:** Manejo de rate limits y errores de API con reintentos
- **Image Processing Errors:** Validación de integridad de imagen
- **Server Errors:** Logging detallado y respuestas sanitizadas

### Error Response Format
```typescript
interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: any;
  };
}
```

## Testing Strategy

### Frontend Testing
- **Unit Tests:** Componentes individuales con Jest y React Testing Library
- **Integration Tests:** Flujo completo de generación de memes
- **Visual Tests:** Validación de renderizado de canvas
- **Accessibility Tests:** Navegación por teclado y screen readers

### Backend Testing
- **Unit Tests:** Funciones de procesamiento de imágenes y validación
- **API Tests:** Endpoints con diferentes escenarios de entrada
- **Integration Tests:** Comunicación con Anthropic API (con mocks)
- **Error Handling Tests:** Validación de todos los casos de error

### Performance Considerations
- **Image Optimization:** Redimensionamiento automático para optimizar rendimiento
- **Caching:** Cache de respuestas de Anthropic para textos similares
- **Lazy Loading:** Carga diferida de componentes no críticos
- **Memory Management:** Limpieza de canvas y objetos URL después del uso

### Security Considerations
- **File Validation:** Validación estricta de tipos MIME y contenido de archivo
- **API Key Protection:** Variables de entorno para credenciales de Anthropic
- **Rate Limiting:** Límites por IP para prevenir abuso
- **Input Sanitization:** Validación y sanitización de todos los inputs