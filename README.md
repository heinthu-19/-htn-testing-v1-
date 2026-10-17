# htn testing V1 — Story-to-Movie edition

An independent Flow-inspired AI filmmaking app. Paste a complete story, let Gemini split it into a sequence of cinematic scene prompts, generate one real AI video clip per scene with Replicate, and assemble the clips into one MP4. Cloudflare API is not used. This is not Google Flow and does not use Google's private APIs.

## How it works

1. Paste the full story in Burmese or English.
2. Choose the approximate movie length and aspect ratio.
3. **Scene ခွဲပြီး Preview** uses Gemini to create an ordered storyboard and generates the first video clip.
4. **ဇာတ်လမ်းမှ Movie ဖန်တီး** generates the remaining clips sequentially, preserving each scene's individual prompt.
5. Review/reorder scenes. Upload a main-character reference and (optionally) a recurring background/location reference before generation. The storyboard creates a character/world continuity guide that is repeated in every scene prompt. On each generated scene, enter the narration and character dialogue, choose a pitch-distinguished dialogue style, and press **ဇာတ်ကြောင်း + Dialogue + Lip-sync**. The app sends the generated dialogue audio and video to the paid Sync Lipsync 2 model; narration is mixed underneath during MP4 assembly. Then press **Export MP4**.

## API keys

Set these in Render → your Web Service → Environment:

- `GEMINI_API_KEY` — required for story-to-scene breakdown. Create a key at https://aistudio.google.com/apikey.
- `REPLICATE_API_TOKEN` — required for image generation, actual video generation and the paid Sync Lipsync 2 lip-sync stage. Create a token at https://replicate.com/account/api-tokens.
- `GEMINI_MODEL` — optional; defaults to `gemini-2.5-flash`.

No Cloudflare API token or account ID is required.
- `PORT` — Render supplies this automatically.

Never place secrets in browser code, HTML, or a public GitHub repository. If a token was previously shared in chat or committed to GitHub, revoke it and create a replacement.

## Important duration and cost note

The configured Replicate model (`minimax/video-01`) generally produces clips around 6 seconds. The MP4 assembler retimes the video to approximately 8 seconds per scene; this stretches the generated motion slightly and is **not** the same as a model natively generating 8 seconds of new motion. Check current model availability, input schema, and pricing before running a long movie. A full movie can cost a significant amount; always preview one scene first.

## Deploy on Render

1. Extract the ZIP.
2. Upload the contents of `Full_AI_StoryMotion/` to the root of a GitHub repository.
3. Create/update a Render Web Service with Build Command `npm install` and Start Command `npm start`.
4. Add both `GEMINI_API_KEY` and `REPLICATE_API_TOKEN` to Environment and redeploy.
5. Open the app, paste the story, generate a preview, then generate the full movie.

## Character and background continuity

- The storyboard builds a shared character bible and world/background bible, then adds them to every clip prompt. This improves consistency but generative video can still drift between clips.
- Upload a clear main-character reference image. The video model receives it as `subject_reference`.
- Optionally upload a background/location reference. If no start frame is supplied, it is used as the first frame for every clip to anchor the environment. This can improve continuity but may make different scenes start from a similar composition.

## Voice and lip-sync notes

- Burmese speech uses best-effort Google Translate TTS with Myanmar-script text. Pronunciation and naturalness can vary; it is not guaranteed to sound like a native professional voice actor. Dialogue voice styles are distinguished using FFmpeg pitch processing, not voice cloning or genuinely different trained speakers.
- Lip-sync uses Replicate `sync/lipsync-2` and may incur additional charges for each scene. Provider model availability, accepted inputs, output quality and pricing can change; test one short scene first. The lip-sync stage has been wired into the app but has not been live-tested with paid credentials in this package.
- Use the scene-level **ဇာတ်ကြောင်း + Dialogue + Lip-sync** button after that scene has a generated video and non-empty dialogue.

## Runtime notes

- Node.js 20+ required.
- MP4 assembly uses `ffmpeg-static`.
- Keep the browser tab open while scenes are generated. The app does not have a persistent database or background queue; a refresh can lose in-browser project state.
- Render temporary filesystems may be ephemeral; download the final MP4 soon after assembly.
- `/api/health` only reports whether keys are configured; it does not make a paid provider test call.

## Local development

```bash
cp .env.example .env
npm install
npm start
```

Open `http://localhost:3000`.
