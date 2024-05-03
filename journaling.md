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

### Asking Questions: More important than Finding Answers

Asking the right questions can be more important
than figuring out their exact answers.

> The mere formulation of a problem
> is far more often essential than its solution,
> which may be merely a matter of mathematical or experimental skill.
> To raise new questions, new possibilities,
> to regard old problems from a new angle,
> requires creative imagination
> and marks real advances in science.
> – Albert Einstein

#### Asking Questions: Why?

The reason asking the right questions can be very valuable is that it
helps us direct our concentration towards the most important aspects of a model.

#### Ask the meta-question

In order to figure out the answer to a given question,
it often pays focus on the derivative:
to explicitly reframe the question as the meta-question
"how could I figure out the answer to this question?"

## Think of Your Future Selves

Be considerate to your future selves
—the versions of you that will exist in the future.
Do now the things that maximize their happiness.

In particular, consider the future versions of you that will exist in:

* 1 hour
* 4 hours
* 12 hours
* 1 day
* 2 days
* 4 days
* 1 week
* 2 weeks
* 1 month
* 1 quarter
* 1 year
* 2 years
* 5 years
* 15 years

When journaling, for each of these identities, ask the question:
"What can I do now that $IDENTITY will appreciate the most?"
The "now" time-frame depends somewhat on the $IDENTITY.

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

    GRAY = colors.HexColor(0xdddddd)
    LIGHT_GRAY = colors.HexColor(0xeeeeee)

    dot_distance = 5  # mm between dots
    dot_size = 0.1  # Diameter of dots in mm
    margin = 14  # Margin around the page in mm

    # Page dimensions in mm (for A4)
    page_width_mm, page_height_mm = A4[0] / mm, A4[1] / mm

    # Adjust start positions to ensure dots are centered with equal margins
    start_x = margin + ((page_width_mm - 2 * margin) % dot_distance) / 2
    start_y = margin + ((page_height_mm - 2 * margin) % dot_distance) / 2

    c.setStrokeColor(GRAY)
    c.setFillColor(GRAY)

    x = start_x
    while x <= page_width_mm - margin:
      y = start_y
      while y <= page_height_mm - margin:
        c.circle(
            x * mm, (page_height_mm - y) * mm, dot_size * mm, fill=1, stroke=1)
        y += dot_distance
      x += dot_distance

    rect_dot_position = 0
    c.setStrokeColor(LIGHT_GRAY)
    for rects in map(len, reversed(['YYYY', 'MM', 'DD', 'XXX'])):
      c.rect((x - (1 + rect_dot_position + rects) * dot_distance) * mm,
             (y - 1 * dot_distance) * mm, (rects * dot_distance) * mm,
             dot_distance * mm)
      rect_dot_position += rects

    c.save()

