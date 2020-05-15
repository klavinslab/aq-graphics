# Aquarium Graphics Library

This library provide methods for building and rendering SVG graphics in Aquarium protocols.

**Example 1**

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

**Example 2**

<img src="/docs/_images/transfer_example_2.png" alt="Transfer Example 1" width="300"/>

```ruby
def show_calls myops, band_choices
    myops.each do |op|
      kit_summary = {}

      this_kit = op.temporary[:input_kit]
      this_item = op.input(INPUT).item
      this_unit = op.temporary[:input_unit]
      this_sample = op.temporary[:input_sample]

      grid = SVGGrid.new(MUTATIONS_LABEL.length, 1, 90, 10)
      categories = []
      PREV_COMPONENTS.each.with_index do |this_component, i|
        alias_label = op.input_refs(INPUT)[i]
        strip_label = self.tube_label(this_kit, this_unit, this_component, this_sample)
        strip = make_strip(strip_label, COLORS[i] + "strip")
        band_choice = this_item.get(make_call_key(alias_label))
        codon_label = label(MUTATIONS_LABEL[i], "font-size".to_sym => 25)
        codon_label.align_with(strip, 'center-bottom')
        codon_label.align!('center-top').translate!(0, 30)
        category = this_item.get(make_call_category_key(alias_label))
        kit_summary[MUTATIONS_LABEL[i]] = {:alias => alias_label, :category => category.to_s, :call => band_choice.to_s}
        tokens = category.split(' ')
        tokens.push("") if tokens.length == 1
        category_label = two_labels(*tokens)
        category_label.scale!(0.75)
        category_label.align!('center-top')
        category_label.align_with(codon_label, 'center-bottom')
        category_label.translate!(0, 10)
        bands = band_choices[band_choice.to_sym][:bands]
        grid.add(strip, i, 0)
        grid.add(codon_label, i, 0)
        grid.add(category_label, i, 0)
        bands.each do |band|
          grid.add(band, i, 0)
        end
      end

      op.associate(:results, kit_summary)
      op.input(INPUT).item.associate(:results, kit_summary)
      op.temporary[:results] = kit_summary

      img = SVGElement.new(children: [grid], boundx: 600, boundy: 350)
      img.translate!(15)
      show do
        refs = op.input_refs(INPUT)
        title "Here is the summary of your results for <b>#{refs[0]}-#{refs[-1]}</b>"
        note display_svg(img)
      end
    end
  end
```
