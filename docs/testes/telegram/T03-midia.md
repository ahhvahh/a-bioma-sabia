# T03 — Mídia

- T03-01: JPEG/PNG válidos e dentro dos limites de foto → sendPhoto.
- T03-02: MP4 suportado → sendVideo.
- T03-03: PDF, imagem de formato incompatível, vídeo incompatível e MIME ausente → sendDocument, se aceito.
- T03-04: foto acima do limite específico → rejeição ou fallback document se dentro do limite total e permitido.
- T03-05: arquivo com 50.000.000 bytes → dentro do limite total (respeitando método).
- T03-06: arquivo com 50.000.001 bytes → rejeitar antes de chamar API.
- T03-07: chunk com 5.000.001 bytes → rejeitar.
- T03-08: download remoto com 20.000.001 bytes → recusar via getFile público.
- T03-09: MIME declarado não é prova de integridade/autorização; impedir execução de conteúdo hostil.
