function fetchWithTimeout(url, options = {}, timeout = 5000) {
  const controller = new AbortController();
  const signal = controller.signal;
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  return fetch(url, { ...options, signal })
    .then((response) => {
      clearTimeout(timeoutId);
      return response;
    })
    .catch((error) => {
      clearTimeout(timeoutId);
      if (error.name === "AbortError") {
        throw new Error("Request timed out");
      }
      throw error;
    });
}

function cmsNest() {
  const allItemPromises = [];

  document.querySelectorAll("[data-cms-nest^='item']").forEach((item) => {
    const link = item.querySelector("[data-cms-nest='link']");
    if (!link) {
      console.warn("CMS Nest: Link not found", item);
      return;
    }

    const href = link.getAttribute("href");
    if (!href) {
      console.warn("CMS Nest: Href attribute not found", link);
      return;
    }

    try {
      const url = new URL(href, window.location.origin);
      if (url.hostname !== window.location.hostname) {
        console.warn("CMS Nest: URL is not on the same domain", url);
        return;
      }

      const dropzones = item.querySelectorAll("[data-cms-nest^='dropzone-']");
      if (dropzones.length === 0) return;

      const itemPromise = fetchWithTimeout(href)
        .then((response) => {
          if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
          }
          return response.text();
        })
        .then((content) => {
          const parsedContent = new DOMParser().parseFromString(content, "text/html");

          dropzones.forEach((dropzone) => {
            const dropzoneNumber = dropzone.getAttribute("data-cms-nest").split("-")[1];
            const target = parsedContent.querySelector(
              `[data-cms-nest='target-${dropzoneNumber}']`
            );

            if (target) {
              dropzone.replaceChildren(target);
            } else {
              console.warn(`CMS Nest: Target-${dropzoneNumber} not found in fetched content`, url);
            }
          });
        })
        .catch((error) => {
          console.error("CMS Nest: Error fetching the link or request timed out:", error);
        });

      allItemPromises.push(itemPromise);
    } catch (error) {
      console.error("CMS Nest: Invalid URL", href, error);
    }
  });

  Promise.all(allItemPromises)
    .then(() => {
      const event = new CustomEvent("cmsNestComplete");
      window.dispatchEvent(event);
    })
    .catch((error) => {
      console.error("CMS Nest: One or more fetch requests failed", error);
    });
}

document.addEventListener("DOMContentLoaded", cmsNest);
