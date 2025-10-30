# generate_readme.py
import json
import os
from pathlib import Path
from urllib.parse import quote_plus

# Load details.json
with open("details.json", "r", encoding="utf-8") as f:
    data = json.load(f)

username = data.get("github_username", "YOUR_USERNAME")
name = data.get("name", username)
title = data.get("title", "")
location = data.get("location", "")
website = data.get("website", "")
email = data.get("email", "")
bio = data.get("bio", "")
featured = data.get("featured_projects", [])
skills = data.get("skills", [])
fun = data.get("fun", {})

# Mapping from skill label -> shields/simple-icons logo name and color (hex)
# Add more mappings as needed
SKILL_MAP = {
    "python": ("python", "3776AB"),
    "fastapi": ("fastapi", "009688"),
    "django": ("django", "092E20"),
    "javascript": ("javascript", "F7DF1E"),
    "typescript": ("typescript", "3178C6"),
    "react": ("react", "61DAFB"),
    "docker": ("docker", "2496ED"),
    "postgres": ("postgresql", "316192"),
    "git": ("git", "F05032"),
    "linux": ("linux", "FCC624"),
    "vscode": ("visualstudiocode", "007ACC"),
    "bootstrap": ("bootstrap", "7952B3"),
    "node": ("node.js", "339933"),
    "nodejs": ("node.js", "339933"),
    "go": ("go", "00ADD8"),
    "redis": ("redis", "DC382D"),
    "mongodb": ("mongodb", "47A248"),
    "html": ("html5", "E34F26"),
    "css": ("css3", "1572B6"),
    "tailwind": ("tailwindcss", "06B6D4"),
    "flask": ("flask", "000000"),
    "aws": ("amazonaws", "FF9900"),
    "firebase": ("firebase", "FFCA28"),
    "figma": ("figma", "F24E1E"),
    "raspberrypi": ("raspberrypi", "C51A4A"),
    "ruby": ("ruby", "CC342D"),
    "php": ("php", "777BB4"),
    "reactnative": ("react", "61DAFB"),
}

def build_badge(skill_label):
    s = skill_label.strip()
    key = s.lower().replace(" ", "").replace("-", "")
    logo, color = None, None
    if key in SKILL_MAP:
        logo, color = SKILL_MAP[key]
    else:
        # try simple heuristic: use key as logo if it exists in simple-icons names (best-effort)
        logo = key
        color = "2b2b2b"
    # Use shields.io badge with logo param (logo uses simple-icons)
    label = quote_plus(s)
    badge = f"https://img.shields.io/badge/-{label}-{color}?style=for-the-badge&logo={quote_plus(logo)}&logoColor=white"
    return badge

# Deduplicate skills while preserving order
seen = set()
unique_skills = []
for s in skills:
    if s.lower() not in seen:
        unique_skills.append(s)
        seen.add(s.lower())

# Build skill image grid (4 per row)
rows = []
per_row = 8  # using 8 per row matches screenshot compactness
for i in range(0, len(unique_skills), per_row):
    row = unique_skills[i:i+per_row]
    rows.append(row)

skill_grid_md = []
skill_grid_md.append("## I have experience developing in 🧰\n")
for row in rows:
    # images separated by space with tiny gap
    imgs = " ".join(
        f"<img src=\"{build_badge(s)}\" alt=\"{s}\" height=\"34\" style=\"margin:4px; border-radius:8px;\" />"
        for s in row
    )
    skill_grid_md.append(imgs + "\n")

skill_section = "\n".join(skill_grid_md)

# Featured projects markdown
proj_lines = []
for p in featured:
    name_p = p.get("name")
    url_p = p.get("url")
    desc_p = p.get("desc", "")
    proj_lines.append(f"- **[{name_p}]({url_p})** — {desc_p}")

projects_md = "\n".join(proj_lines) if proj_lines else "- _No featured projects yet_"

# Stats images (live)
stats_md = f"""
<!-- STATS -->
<p align="center">
  <img alt="{name}'s GitHub stats" src="https://github-readme-stats.vercel.app/api?username={username}&show_icons=true&theme=radical" />
  &nbsp;
  <img alt="Top languages" src="https://github-readme-stats.vercel.app/api/top-langs?username={username}&layout=compact&theme=radical" />
</p>
"""

# Assemble README
readme = f"""<!--
  Generated README — update details.json and re-run generate_readme.py
  Paste the resulting README.md into a repo named exactly: {username}
-->

<div align="center">
  <h1>Hi 👋, I'm {name}</h1>
  <p><em>{title}</em></p>
</div>

<p align="center">
  🌍 I'm based in {location} •
  <a href="{website}" target="_blank">Personal Website</a> •
  📫 {email}
</p>

---

## About
{bio}

---

{stats_md}

---

## 🔭 Featured Projects

{projects_md}

---

{skill_section}

---

## 📫 Contact
- Email: {email}  
- Website: {website}

---

## ⚡ Fun
- 🔭 Working on: {fun.get('working_on','')}
- 🎧 Listening to: {fun.get('listening_to','')}

<p align="center">Made with ❤️ — <strong>{name}</strong></p>
"""

# Output README.md
with open("README.md", "w", encoding="utf-8") as f:
    f.write(readme)

print("README.md generated. Inspect and commit it to a repo named exactly:", username)
