# Journaling

Journaling is, for me, a very useful mechanism to introspect; it has
enabled me to identify ideas that were hidden in plain sight and come to very
useful insights that have accelerated my career path.

## Ideas Hidden in Plain Sight

Many good ideas remain hidden in plain sight for a very long time.

## Asking questions

A useful technique when journaling
is focusing on writing down a tree (or sequence) of questions,
where explicitly stating each question
prompts child subquestions
that tend to be easier to answer
and/or to advance my model of the reality under analysis
in directions that can yield further insights.

## Think of Your Future Self

Be considerate to your future self (the version of you that will exist in the future): do now the things that will make them the happiest.

In particular, consider the version of you that will exist:

* 1 hour from now
* 4 hours from now
* 12 hours from now
* 1 day from now
* 2 days from now
* 4 days from now
* 1 week from now
* 2 weeks from now
* 1 month from now
* 1 quarter from now
* 1 year from now
* 2 years from now
* 5 years from now
* 15 years from now

When journaling, for each of these identities, ask the question:
"What can I do now that $IDENTITY will appreciate the most?"

## Format

I found that using a fountain pen and paper for journaling
works better *for me* than other approaches I've tried.

The difference isn't huge:
all of these digital formats are almost as good.
So I can't argue very strongly one way or another.

### Digital

I tried various digital approaches for journaling over the years:

* A Google Docs file.

* Markdown files in
  [my Zettelkasten](https://github.com/alefore/weblog/blob/master/zettelkasten.md)
  that I edit through
  [my text editor](https://github.com/alefore/edge).

* A Supernote Tablet with a Lamy digital pen.


### Linux: Generate background for notes

I prefer to write over paper with a grid of dots.

I've experimented with various parameters such as:

* Distance between dots.

* Dot size.

* Dot color (weight).

* Margin.

I implemented my preferences as a Python + ReportLab script.
The script generates a `background.pdf` PDF file with a single-page,
which I can print as often as I want.

The top of the page has a small table where I write the date
(YYYY-MM-DD) and page ID (within the date).

File `~/bin/pdf-notes-background` (mode 755):

    #!/home/alejo/local/bin/python3
    from reportlab.lib.pagesizes import A4
    from reportlab.pdfgen import canvas
    from reportlab.lib.units import mm
    from reportlab.lib import colors

    c = canvas.Canvas("background.pdf", pagesize=A4)

    dot_distance = 5  # mm between dots
    dot_size = 0.1  # Diameter of dots in mm
    margin = 14  # Margin around the page in mm

    # Page dimensions in mm (for A4)
    page_width_mm, page_height_mm = A4[0] / mm, A4[1] / mm

    # Adjust start positions to ensure dots are centered with equal margins
    start_x = margin + ((page_width_mm - 2 * margin) % dot_distance) / 2
    start_y = margin + ((page_height_mm - 2 * margin) % dot_distance) / 2

    c.setStrokeColor(colors.HexColor(0xeeeeee))  # Set fill color to light gray
    c.setFillColor(colors.HexColor(0xeeeeee))  # Set fill color to light gray

    x = start_x
    while x <= page_width_mm - margin:
      y = start_y
      while y <= page_height_mm - margin:
        c.circle(
            x * mm, (page_height_mm - y) * mm, dot_size * mm, fill=1, stroke=1)
        y += dot_distance
      x += dot_distance

    rect_dot_position = 0
    c.setStrokeColor(colors.HexColor(0xeeeeee))  # Set fill color to light gray
    for rects in map(len, reversed(['YYYY', 'MM', 'DD', 'XXX'])):
      c.rect((x - (1 + rect_dot_position + rects) * dot_distance) * mm,
             (y - 1 * dot_distance) * mm, (rects * dot_distance) * mm,
             dot_distance * mm)
      rect_dot_position += rects

    c.save()

