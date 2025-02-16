---
layout: "post"
title: "Favorite Quote"
date: 2025-01-10 11:11:11 +/-0900
categories: Favorite
tags: quote
excerpt: "Share your favorite quotes"
---

<div class="header-container">
  <h2>Favorite Quotes</h2>
  <button id="refresh-btn">🔄 Refresh</button>
</div>

<p id="loading-text">Loading<span id="dots">.</span></p> <!-- Animated Loading Message -->

<div id="thread-container"></div>

<h2>Submit Your Favorite Quote</h2>
<div class="form-container">
  <iframe 
    src="https://docs.google.com/forms/d/e/1FAIpQLSeagOaDNvbzyDgos0zdQZMNfuZtAMySpaIaILZ5C3_iJFSfPg/viewform?embedded=true" 
    width="100%" height="700" frameborder="0" marginheight="0" marginwidth="0">
    Loading...
  </iframe>
</div>

<script>

let dotCount = 1;
const dotsElement = document.getElementById("dots");
const loadingText = document.getElementById("loading-text");

function animateDots() {
  dotCount = (dotCount % 3) + 1;
  dotsElement.textContent = ".".repeat(dotCount);
}
const dotInterval = setInterval(animateDots, 500);

async function fetchGoogleSheet() {
    const url = "https://script.google.com/macros/s/AKfycbw2_Dm3rsmvckcDalfLdGrQpZDtCl92JiRN8s_mwW9bIEud1pNS2AoXSKXExVtfkNhR/exec";
    try {
        const response = await fetch(url);
        const data = await response.json();
        const threadContainer = document.getElementById("thread-container");
        threadContainer.innerHTML = "";
        let posts = data.slice(1).reverse();
        let visiblePosts = posts.slice(0, 10);

        function createPost(row) {
            let post = document.createElement("div");
            post.classList.add("post");

            let timestamp = document.createElement("p");
            timestamp.classList.add("timestamp");
            timestamp.textContent = convertToKST(row.Timestamp);

            let quote = document.createElement("h3");
            const fullQuote = row.Quote;
            const shortQuote = fullQuote.length > 500 ? fullQuote.substring(0, 500) + "..." : fullQuote;

            let quoteContent = document.createElement("span");
            quoteContent.classList.add("short-text");
            quoteContent.textContent = shortQuote;

            let fullContent = document.createElement("span");
            fullContent.classList.add("full-text", "hidden");
            fullContent.textContent = fullQuote;

            if (fullQuote.length > 500) {
                let readMoreBtn = document.createElement("button");
                readMoreBtn.classList.add("read-more-btn");
                readMoreBtn.textContent = "Read More";

                readMoreBtn.addEventListener('click', function() {
                    quoteContent.classList.add('hidden');
                    fullContent.classList.remove('hidden');
                    this.remove();
                });

                quote.appendChild(quoteContent);
                quote.appendChild(fullContent);
                quote.appendChild(readMoreBtn);
            } else {
                quote.appendChild(quoteContent);
            }

            let speaker = document.createElement("p");
            speaker.textContent = row.Speaker.length > 100 ? row.Speaker.substring(0, 100) + "..." : row.Speaker;

            let email = document.createElement("p");
            email.classList.add("email");
            email.textContent = row.Email;

            post.appendChild(timestamp);
            post.appendChild(quote);
            post.appendChild(speaker);
            post.appendChild(email);
            return post;
        }

        visiblePosts.forEach(row => threadContainer.appendChild(createPost(row)));

        if (posts.length > 10) {
            let showMoreBtn = document.createElement("button");
            showMoreBtn.textContent = "Show More Posts";
            showMoreBtn.classList.add("show-more-btn");
            threadContainer.appendChild(showMoreBtn);
            showMoreBtn.addEventListener("click", function() {
                posts.slice(10).forEach(row => threadContainer.appendChild(createPost(row)));
                this.remove();
            });
        }

        loadingText.style.display = "none";
        clearInterval(dotInterval); 
    } catch (error) {
        console.error("Error fetching Google Sheet:", error);
        loadingText.textContent = "Failed to load data!";
        clearInterval(dotInterval);
    }
}


document.getElementById("refresh-btn").addEventListener("click", function() {
    let refreshBtn = this;
    let dotCount = 0;
    refreshBtn.disabled = true;
    let dotInterval = setInterval(() => {
        dotCount = (dotCount % 3) + 1;
        refreshBtn.textContent = "🔄 Refreshing" + ".".repeat(dotCount);
    }, 500);
    fetchGoogleSheet().then(() => {
        clearInterval(dotInterval);
        refreshBtn.textContent = "🔄 Refresh";
        refreshBtn.disabled = false;
    });
});

function convertToKST(utcDateStr) {
    const utcDate = new Date(utcDateStr);
    utcDate.setHours(utcDate.getHours() + 9);
    return utcDate.toISOString().replace("T", " ").split(".")[0];
}

document.addEventListener("DOMContentLoaded", fetchGoogleSheet);
</script>

<style>
  /* Thread Container */
  #thread-container {
    max-width: 800px;
    margin: 20px auto;
    padding: 10px;
    display: flex;
    flex-direction: column;
  }

  /* Individual Post Box */
  .post {
    border: 1px solid #ccc;
    padding: 15px;
    margin-bottom: 10px;
    border-radius: 5px;
  }

  /* Timestamp Style */
  .timestamp {
    font-size: 12px;
    color: #777;
    margin-bottom: 5px;
    line-height: 1.2;
  }

  /* Quote (Title) Styling */
  h3 {
    margin-top: 2px !important;
    margin-bottom: 8px !important;
    line-height: 1.2;
  }

  /* Paragraphs (Speaker, Email) */
  p {
    margin: 5px 0;
  }

  /* User Email Style */
  .email {
    font-size: 12px;
    color: #999;
    margin-top: 8px;
  }

  /* Header Container for Title and Refresh Button */
  .header-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    max-width: 800px;
    margin: 0 auto;
    padding: 10px 0;
  }

  /* Refresh Button Styling */
  #refresh-btn {
    padding: 5px 10px;
    font-size: 14px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background 0.3s;
    min-width: 120px;
    text-align: center;
  }

  #refresh-btn:hover {
    background-color: #0056b3;
  }

  #refresh-btn:disabled {
    background-color: #6c757d;
    cursor: not-allowed;
  }

  /* Hidden Elements */
  .hidden {
    display: none;
  }

  /* Read More Button */
  .read-more-btn {
    margin-top: 5px;
    padding: 5px 10px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 12px;
    transition: background 0.3s;
  }

  .read-more-btn:hover {
    background-color: #0056b3;
  }

  /* Show More Posts Button */
  .show-more-btn {
    margin-top: 10px;
    padding: 8px 12px;
    background-color: #28a745;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 14px;
    transition: background 0.3s;
  }

  .show-more-btn:hover {
    background-color: #218838;
  }
</style>
