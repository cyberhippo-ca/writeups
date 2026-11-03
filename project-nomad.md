# An Offline Knowledge and AI Box with Project NOMAD

*Posted October 28, 2026 · Also on [cyberhippo.ca](https://cyberhippo.ca/writeups/project-nomad)*

I set up [Project NOMAD](https://www.projectnomad.us/) on an Acer Nitro V 15 to have a box that works with no internet: offline Wikipedia, maps, medical and reference libraries, and a local AI model I can ask questions. NOMAD is an open-source project from Crosstalk Solutions that bundles all of this behind one web dashboard.

**Result:** NOMAD running in WSL2 on Windows, with Wikipedia, maps of Canada, reference and education content, and Qwen 3.5 9B running locally through Ollama.

## Hardware

| Part   | Spec                       | Why it matters                                                           |
|--------|----------------------------|--------------------------------------------------------------------------|
| Laptop | Acer Nitro V 15            | x86 with an NVIDIA GPU, which is what NOMAD's AI tools expect            |
| GPU    | RTX 5060 laptop, 8 GB VRAM | The limit that decides which AI models run fast                          |
| RAM    | 32 GB                      | Room to offload bigger models to system memory, at the cost of speed     |
| OS     | Windows                    | NOMAD is built for Debian-based Linux, so this shaped the install choice |

## 1. Choosing how to install it

NOMAD is built for Debian-based Linux, and the laptop runs Windows. The options were:

| Option              | Upside                                                                          | Downside                             |
|---------------------|---------------------------------------------------------------------------------|--------------------------------------|
| Full Ubuntu install | Officially recommended, best performance                                        | Wipes Windows                        |
| **WSL2 on Windows** | Keeps Windows, no dual boot ([guide](https://www.projectnomad.us/install/wsl2)) | Community-supported, slightly slower |
| Virtual machine     | Isolated                                                                        | Not supported by the project         |

I went with **WSL2** so the laptop stays usable as a Windows machine. After the install, NOMAD's **Easy Setup** wizard in the browser picks which content to download.

## 2. Content

- **Offline Wikipedia**
- **Maps** of Canada (see the next section)
- **Medical, reference and education** libraries
- **AI chat** through Ollama

Storage adds up fast: full English Wikipedia with images is over 100 GB on its own, so it's worth planning disk space before ticking everything.

## 3. The map download that broke the wizard

Every time I tried to download maps, the wizard sat on "estimating the map download size" and then failed with a **500 error**. The rest of NOMAD worked; only this step failed.

The cause was my request: **all of Canada at zoom level 15**.

Map tiles grow exponentially with zoom. Each zoom level splits every tile into four, so zoom 15 has about **1,000 times** as many tiles as zoom 10 over the same area (4<sup>5</sup> = 1,024). Zoom 15 is street-level detail, and Canada is the second-largest country in the world. The size estimate couldn't finish, and the backend returned a 500 instead of a number.

**Fix:** I dropped the zoom level for the Canada-wide map, and the estimate and download completed. Street-level zoom only makes sense for a small area, like a single city.

## 4. Choosing an AI model

With 8 GB of VRAM, the practical range is models up to about 8 to 9 billion parameters, quantized so they fit entirely on the GPU. Bigger models (14B to 32B) can run by spilling into the 32 GB of system RAM, but much more slowly.

I picked **Qwen 3.5 9B**, a common recommendation for 8 GB cards. Rankings in this space change month to month, so I'll revisit the choice rather than treat it as settled.

Other picks depend on the job:

- **Tool use and agents:** smaller models that are reliable at structured output can beat larger ones, because one malformed response breaks an agent loop
- **Searching my own documents:** a separate embedding model alongside the chat model
- **Coding:** a coding-tuned 7B model fits; the strong agentic coding models are too big for 8 GB

## What I learned

- **The GPU's memory is the real limit.** VRAM decides which models run well, more than the CPU or system RAM.
- **Read the error in context.** "Estimating size" then a 500 pointed at the estimate step, not the download or my connection.
- **Zoom levels are exponential.** Five zoom levels is a thousandfold difference in tiles. Match the detail to the area.
- **Pick the install path for how you use the machine.** WSL2 traded a little performance for keeping Windows.

## Next steps

- Make it always on: start NOMAD and Ollama at boot, and set a battery charge limit for a laptop that's plugged in all the time
- Reach it from my other devices over my WireGuard VPN, without exposing it to the internet
- Keep Ollama's API off the network, or limit it to specific devices, since it has no login
- Add more detailed maps for my local area

## References

- [Project NOMAD](https://www.projectnomad.us/) and its [GitHub repo](https://github.com/Crosstalk-Solutions/project-nomad)
- [Project NOMAD: WSL2 install guide](https://www.projectnomad.us/install/wsl2)
- [Microsoft: install WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Ollama](https://ollama.com/)
- [OpenStreetMap: zoom levels](https://wiki.openstreetmap.org/wiki/Zoom_levels)

---

[All write-ups](README.md) · [cyberhippo.ca](https://cyberhippo.ca)
