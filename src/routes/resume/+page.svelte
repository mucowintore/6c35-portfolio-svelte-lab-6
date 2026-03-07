<svelte:head>
  <title>Resume</title>
</svelte:head>

<div class="resume-page">
  <main>
    <h1>Resume</h1>
    <header>
      <p><strong>Muco</strong> - CS Student · Cambridge, MA · email@example.com</p>
    </header>

    <section>
      <h2>Education</h2>
      <article class="education-item">
        <h3>Massachusetts Institute of Technology</h3>
        <p>B.S. in Computer Science</p>
        <p><time datetime="2022-09">Sep 2022</time> - <time datetime="2026-06">Jun 2026</time></p>
        <br />
        <p>Relevant coursework: Web Development, Systems, Linear Algebra</p>
      </article>
    </section>

    <section>
      <h2>Experience</h2>

      <article>
        <header>
          <h3>Software Engineering Intern - ACME Corp</h3>
          <p><time datetime="2025-06">Jun 2025</time> - <time datetime="2025-08">Aug 2025</time></p>
        </header>
        <ul>
          <li>Built a small internal tool using HTML/CSS</li>
          <li>Improved page load time by 20%</li>
        </ul>
      </article>

      <article>
        <header>
          <h3>Research Assistant - ACME Lab</h3>
          <p><time datetime="2024-09">Sep 2024</time> - <time datetime="2025-05">May 2025</time></p>
        </header>
        <ul>
          <li>Prototyped a learning app and ran user tests</li>
        </ul>
      </article>
    </section>

    <section>
      <h2>Skills</h2>
      <ul>
        <li>Languages: Python, JavaScript</li>
        <li>Web: HTML, CSS</li>
        <li>Tools: Git, GitHub</li>
      </ul>
    </section>
  </main>
</div>
