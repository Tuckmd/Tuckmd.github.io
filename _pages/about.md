---
permalink: /
title: "Michael Tuck"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

<div style="position: relative; width: 100%; max-width: 700px; margin: 2rem auto;">
  <div id="slideshow" style="overflow: hidden; border-radius: 8px; border: 0.5px solid #ddd;">
    <div id="slides" style="display: flex; transition: transform 0.4s ease;">
    </div>
  </div>
  <button onclick="changeSlide(-1)" style="position: absolute; left: -20px; top: 50%; transform: translateY(-50%); border-radius: 50%; width: 36px; height: 36px; cursor: pointer;">&#8592;</button>
  <button onclick="changeSlide(1)" style="position: absolute; right: -20px; top: 50%; transform: translateY(-50%); border-radius: 50%; width: 36px; height: 36px; cursor: pointer;">&#8594;</button>
  <div id="dots" style="text-align: center; margin-top: 10px;"></div>
  <p id="caption" style="text-align: center; font-size: 13px; color: #666; margin-top: 8px; min-height: 20px;"></p>
</div>

<script>
const slides = [
  { src: "/images/thesis_graphical_abstract.png", caption: "Graphical abstract for my PhD work" },
  { src: "/images/bordeaux_instrument.jpg", caption: "Much of the work during my PhD in Bordeaux was conducted on the Orbitrap Qexactive combined with the TransMIT AP-SMALDI5 ionization source" },
  { src: "/images/frontiers_multimodal.png", caption: "Workflow for multimodal imaging data integration and registration (Frontiers 2021)" },
  { src: "/images/OR_pic.jpg", caption: "Dr. Shanneik and I conducting mobile medical mass spectrometry in the Operating Room" },
  { src: "/images/graphical_abstract_mmi.png", caption: "Multimodal imaging combined with vibrational spectroscopy (ACS 2020)" },
  { src: "/images/prize_pic.jpg", caption: "The Michael E. DeBakey Department of Surgery Innovation Scholar Award delivered by my PI, Dr. Livia Eberlin" },
];

let current = 0;
const track = document.getElementById('slides');
const dotsEl = document.getElementById('dots');
const captionEl = document.getElementById('caption');

slides.forEach((s, i) => {
  const img = document.createElement('img');
  img.src = s.src;
  img.style = "min-width: 100%; max-height: 400px; object-fit: contain; background: #f5f5f5;";
  track.appendChild(img);
  const dot = document.createElement('span');
  dot.style = "display: inline-block; width: 8px; height: 8px; border-radius: 50%; background: #ccc; margin: 0 4px; cursor: pointer;";
  dot.onclick = () => goTo(i);
  dotsEl.appendChild(dot);
});

function goTo(n) {
  current = (n + slides.length) % slides.length;
  track.style.transform = `translateX(-${current * 100}%)`;
  captionEl.textContent = slides[current].caption;
  document.querySelectorAll('#dots span').forEach((d, i) => d.style.background = i === current ? '#378ADD' : '#ccc');
}

function changeSlide(dir) { goTo(current + dir); }

goTo(0);
</script>

I am an analytical chemist and biomedical researcher specializing in mass spectrometry imaging and spatial metabolomics. I am currently a postdoctoral associate in Livia Eberlin's group at Baylor College of Medicine, where my work focuses on developing and applying spatial and medical mass spectrometry approaches to study metabolism directly in tissue, with applications in surgical oncology. My prior work focused extensively on infectious disease, particularly tuberculosis.

I have over a decade of experience in mass spectrometry imaging, including MALDI and DESI, and have developed quantitative workflows for measuring drugs and metabolites in complex biological systems. I completed my PhD at Université de Bordeaux under the direction of Nicolas Desbenoit, where I developed methods for the simultaneous quantification of multiple anti-tuberculosis drugs in tissue. I began my research career at Vanderbilt University in the Mass Spectrometry Resource Center led by Richard Caprioli.

My current research integrates real-time metabolic flux analysis and spatial biology to better understand disease processes and support translational and clinical applications.

## Work and Resources

Thank you for your interest in my work. On this site you can find links and descriptions of my [publications](/publications/), a [map of my presentations](/talkmap.html), and other [CV items](/cv/). I also build and post [research analytics tools](/posts/2024/01/author-analytics/) and occasionally write about research methodology. You can find these in the [Blog](/year-archive/) section.

If you are interested in collaborating on a project, please feel free to reach out. Or, kill some time with this mass spec themed browser game I made!

<div style="margin: 1.5rem 0; display: flex; gap: 16px; align-items: center; padding: 14px 18px; border: 1px solid #333; border-radius: 8px; background: rgba(0,0,20,0.4);">
  <img src="/images/rotating_saddle_clean.gif" style="width: 80px; height: 80px; object-fit: cover; border-radius: 6px; flex-shrink: 0;" />
  <div>
    <strong style="font-size: 1.05em;">🎮 Ion Runner</strong><br>
    <small>A browser game based on quadrupole ion physics. Control the RF/DC field and keep your ion on a stable trajectory.<br>
    <a href="/posts/2026/05/ion_runner/">Play now →</a></small>
  </div>
</div>
