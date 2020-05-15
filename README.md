# Aquarium Graphics Library

This library provide methods for building and rendering SVG graphics in Aquarium protocols.

<img src="/docs/_images/transfer_example_1.png" alt="Transfer Example 1" width="300"/>

```ruby
  def make_transfer(from, to, spacing, top_label, bottom_label)
    to.align_with(from, 'center-right').align!('center-left')
    to.translate!(spacing)
    arrow = make_arrow(from, to, top_label, bottom_label)
    elements = [arrow, from, to]
    puts elements.map {|e| Vector[e.x, e.y] + e.get_abs_anchor('center-right')}
    max_x = elements.map {|e| (Vector[e.x, e.y] + e.get_abs_anchor('center-right'))[0]}.max
    max_y = elements.map {|e| (Vector[e.x, e.y] + e.get_abs_anchor('center-bottom'))[1]}.max
    svg = SVGElement.new(
        children: elements,
        boundx: 700,
        boundy: 300,
        )
    svg.translate!(20)
  end
  
  show do
    title "Collect clear fluid from #{op.ref("sample tube 1", true)} and add to empty tube #{op.ref("sample tube 2", true).bold}"
    note "Use P1000 pipette set to 900uL"
    note "Point tip away from the dark portion and pipette up slowly."
    warning "Do not tilt the magnetic rack."
    tube = make_tube(opentube, ["empty", "tube"], op.tube_label("sample tube 2", true))
    img = make_transfer(sep_1_diagram, tube, 250, "900uL", "(P1000 pipette)")
    img.translate!(0, -100)
    note display_svg(img, 0.75)
  end
```
