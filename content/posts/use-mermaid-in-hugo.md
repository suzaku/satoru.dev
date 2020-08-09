+++
title = "Using Mermaid in Hugo"
date = 2020-08-09T20:47:40+08:00
draft = false
tags = []
categories = []
mermaid = true
+++

## Create a [shortcode](https://gohugo.io/content-management/shortcodes/) for Mermaid

1. Create a `layouts/shortcodes` directory in your theme directory. 
1. Inside `layouts/shortcodes`, create a file named `mermaid.html`. The filename is the same as the shortcode name we are going to use. Paste the following content into the file:
    ```html
    <div class="mermaid">
        {{.Inner}}
    </div>
    ```

1. Load MermaidJS on posts that requires it. Find somewhere in your theme that's suitable for adding a `<scritp>` and add the following snippet:

   ```html
    {{ if (.Params.mermaid) }}
    <!-- MermaidJS support -->
    <script async src="https://unpkg.com/mermaid@8.2.3/dist/mermaid.min.js"></script>
    {{ end }}
   ```
## Use the mermaid shortcode to create a graph
1. In a post you want to use Mermaid, you need to turn the `mermaid` option on to load the required script. For example, the configuration for this post looks like:
   ```yaml
    +++
    title = "Using Mermaid in Hugo"
    date = 2020-08-09T20:47:40+08:00
    mermaid = true
    +++
   ```
1. Use the short code.
   For example:

   ```html
    {{</* mermaid */>}}
    sequenceDiagram
        participant Alice
        participant Bob
        Alice->>John: Hello John, how are you?
        loop Healthcheck
            John->John: Fight against hypochondria
        end
        Note right of John: Rational thoughts <br/>prevail...
        John-->Alice: Great!
        John->Bob: How about you?
        Bob-->John: Jolly good!
    {{</* /mermaid */>}}
   ```

   generates the following graph:

{{< mermaid >}}
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail...
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!
{{< /mermaid >}}