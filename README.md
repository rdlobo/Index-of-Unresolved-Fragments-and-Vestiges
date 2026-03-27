---
---
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{{ site.title | default: "Flavor Text Archive" }}</title>
  <style>
    body { margin: 0; font-family: Arial, sans-serif; background: #0f0f14; color: #e6e6e6; }
    header { padding: 2rem; text-align: center; border-bottom: 1px solid #333; }
    h1 { margin: 0; font-weight: 400; letter-spacing: 1px; }
    #filters { padding: 1rem; display: flex; flex-wrap: wrap; gap: 0.5rem; justify-content: center; border-bottom: 1px solid #333; }
    .tag { padding: 0.4rem 0.8rem; border: 1px solid #555; border-radius: 20px; cursor: pointer; font-size: 0.85rem; }
    .tag.active { background: #e6e6e6; color: #0f0f14; }
    #entries { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 1rem; padding: 1.5rem; }
    .entry { border: 1px solid #333; padding: 1rem; background: #16161d; display: flex; flex-direction: column; gap: 0.75rem; }
    .entry img { max-width: 100%; height: auto; border: 1px solid #333; }
    .entry-text { font-style: italic; line-height: 1.4; }
    .entry-source { font-size: 0.75rem; opacity: 0.6; }
    .entry-tags { display: flex; flex-wrap: wrap; gap: 0.4rem; }
    .entry-tags span { font-size: 0.75rem; opacity: 0.7; }
  </style>
</head>
<body>
  <header>
    <h1>{{ site.title | default: "Flavor Text Archive" }}</h1>
    <p>{{ site.description | default: "Fragments of systems that may or may not exist." }}</p>
    {{ site.fragments | size }} items indexed.
  </header>

  <div id="filters"></div>
  <div id="entries"></div>

  <script>
    // ========================
    // JEKYLL COLLECTION SOURCE
    // ========================

    const data = [
      {% for item in site.fragments %}
      {
        text: {{ item.text | jsonify }},
        tags: {{ item.tags | jsonify }},
        image: {{ item.image | jsonify }},
        source: {{ item.source | jsonify }}
      }{% unless forloop.last %},{% endunless %}
      {% endfor %}
    ];

    const entriesContainer = document.getElementById("entries");
    const filtersContainer = document.getElementById("filters");

    let activeTags = new Set();

    function getAllTags() {
      const tagSet = new Set();
      data.forEach(item => (item.tags || []).forEach(tag => tagSet.add(tag)));
      return Array.from(tagSet);
    }

    function renderFilters() {
      const tags = getAllTags();

      tags.forEach(tag => {
        const btn = document.createElement("div");
        btn.className = "tag";
        btn.textContent = tag;

        btn.onclick = () => {
          if (activeTags.has(tag)) {
            activeTags.delete(tag);
          } else {
            activeTags.add(tag);
          }
          render();
        };

        if (activeTags.has(tag)) btn.classList.add("active");

        filtersContainer.appendChild(btn);
      });
    }

    function renderEntries() {
      entriesContainer.innerHTML = "";

      data
        .filter(item => {
          if (activeTags.size === 0) return true;
          return [...activeTags].every(tag => (item.tags || []).includes(tag));
        })
        .forEach(item => {
          const div = document.createElement("div");
          div.className = "entry";

          if (item.image) {
            const img = document.createElement("img");
            img.src = ("{{ site.baseurl }}" || "") + item.image;
            div.appendChild(img);
          }

          const text = document.createElement("div");
          text.className = "entry-text";
          text.textContent = item.text;
          div.appendChild(text);

          if (item.source) {
            const source = document.createElement("div");
            source.className = "entry-source";
            source.textContent = "Image source is " + item.source;
            div.appendChild(source);
          }

          const tagsDiv = document.createElement("div");
          tagsDiv.className = "entry-tags";

          (item.tags || []).forEach(tag => {
            const span = document.createElement("span");
            span.textContent = "#" + tag;
            tagsDiv.appendChild(span);
          });

          div.appendChild(tagsDiv);
          entriesContainer.appendChild(div);
        });
    }

    function render() {
      filtersContainer.innerHTML = "";
      renderFilters();
      renderEntries();
    }

    render();
  </script>
</body>
</html>
