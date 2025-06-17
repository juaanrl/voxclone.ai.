// pages/index.js
import { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Textarea } from "@/components/ui/textarea";
import { Input } from "@/components/ui/input";

export default function Home() {
  const [audioFile, setAudioFile] = useState(null);
  const [videoFile, setVideoFile] = useState(null);
  const [text, setText] = useState("");
  const [resultUrl, setResultUrl] = useState(null);
  const [loading, setLoading] = useState(false);
  const [email, setEmail] = useState("");
  const [registered, setRegistered] = useState(false);
  const [charLimit, setCharLimit] = useState(300);

  const handleAudioUpload = (e) => {
    setAudioFile(e.target.files[0]);
  };

  const handleVideoUpload = (e) => {
    setVideoFile(e.target.files[0]);
  };

  const handleTextChange = (e) => {
    const value = e.target.value;
    if (!registered && value.length > charLimit) return;
    setText(value);
  };

  const handleEmailRegister = async () => {
    if (!email.includes("@")) {
      alert("Introduce un correo válido");
      return;
    }

    try {
      await fetch("/api/register", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email }),
      });

      setRegistered(true);
      alert("¡Gracias por registrarte! Te enviaremos novedades por correo.");
    } catch (error) {
      console.error("Error al registrar email:", error);
      alert("Hubo un error al registrar el correo.");
    }
  };

  const handleGenerate = async () => {
    if (!text) {
      alert("Por favor, escribe un texto.");
      return;
    }

    setLoading(true);

    try {
      const voiceId = "21m00Tcm4TlvDq8ikWAM";
      const apiKey = "sk_df4921591e900953db1a7eaaf9d0d29e3aceed42865c80f8";

      const response = await fetch(`https://api.elevenlabs.io/v1/text-to-speech/${voiceId}`, {
        method: "POST",
        headers: {
          "xi-api-key": apiKey,
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          text,
          model_id: "eleven_monolingual_v1",
          voice_settings: {
            stability: 0.5,
            similarity_boost: 0.75,
          },
        }),
      });

      if (!response.ok) throw new Error("Error al generar audio");

      const blob = await response.blob();
      const audioUrl = URL.createObjectURL(blob);
      setResultUrl(audioUrl);

      // guardar actividad en BD
      await fetch("/api/log", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email: registered ? email : null, text }),
      });

    } catch (err) {
      alert("Error generando audio con ElevenLabs");
      console.error(err);
    }

    setLoading(false);
  };

  return (
    <div className="min-h-screen bg-gray-100 p-6">
      <div className="max-w-3xl mx-auto">
        <h1 className="text-4xl font-bold text-center mb-6">Clona tu voz y rostro con IA</h1>

        {!registered && (
          <Card className="mb-4">
            <CardContent className="p-4 space-y-2">
              <p className="font-medium">¿Quieres desbloquear funciones sin límite?</p>
              <Input
                type="email"
                placeholder="Introduce tu correo electrónico"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
              />
              <Button className="w-full" onClick={handleEmailRegister}>
                Registrarse gratis
              </Button>
              <p className="text-xs text-gray-500">
                Sin registro, solo puedes generar textos de hasta {charLimit} caracteres.
              </p>
            </CardContent>
          </Card>
        )}

        <Card className="mb-4">
          <CardContent className="p-4 space-y-4">
            <div>
              <label className="block font-semibold mb-1">1. Sube tu grabación de voz</label>
              <Input type="file" accept="audio/*" onChange={handleAudioUpload} />
            </div>
            <div>
              <label className="block font-semibold mb-1">2. (Opcional) Sube un video con tu rostro</label>
              <Input type="file" accept="video/*" onChange={handleVideoUpload} />
            </div>
            <div>
              <label className="block font-semibold mb-1">3. Escribe el texto que quieres leer</label>
              <Textarea
                rows={4}
                value={text}
                placeholder="Hola, soy una IA leyendo este texto..."
                onChange={handleTextChange}
              />
              {!registered && (
                <p className="text-xs text-gray-500">Máximo {charLimit} caracteres sin registrarse.</p>
              )}
            </div>
            <Button onClick={handleGenerate} disabled={loading || (!registered && text.length > charLimit)} className="w-full">
              {loading ? "Generando..." : "Generar locución/video"}
            </Button>
          </CardContent>
        </Card>

        {resultUrl && (
          <Card className="mb-4">
            <CardContent className="p-4">
              <h2 className="text-xl font-semibold mb-2">🔊 Resultado generado</h2>
              <audio controls className="w-full">
                <source src={resultUrl} type="audio/mpeg" />
                Tu navegador no soporta audio HTML5.
              </audio>
            </CardContent>
          </Card>
        )}

        <p className="text-sm text-gray-600 text-center">
          Esta versión ya utiliza ElevenLabs para generar audio con IA desde tu texto.
        </p>
      </div>
    </div>
  );
}
