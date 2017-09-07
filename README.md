# WFP Image Map Feature Documentation

## Creating the SVG map file

1. You will need a SVG file with a `<path>` element for each clickable region that you want to present in your form. For an example, see [US_MAP.svg](https://github.com/kobotoolbox/wfp-svg-map-documentation/blob/master/US_MAP.svg?short_path=ab2ce7a);
1. Consider how you will construct the [`choices` worksheet](http://xlsform.org/#choices-worksheet) of your XLSForm. The `id` attribute of each `<path>` element in the SVG file must correspond to a value in the `name` column of your `choices` worksheet;
1. Remember that SVG is a plain-text format; it's possible to modify any SVG file as needed using your favorite text editor.

## Preparing the XLSForm

### Example `choices` sheet

| list name | name | label       |
|-----------|------|-------------|
| state     | AK   | Alaska      |
| state     | HI   | Hawaii      |
| state     | AL   | Alabama     |
| state     | AR   | Arkansas    |
| state     | AZ   | Arizona     |
| state     | CA   | California  |
| state     | CO   | Colorado    |
| state     | CT   | Connecticut |
| state     | DE   | Delaware    |
| state     | FL   | Florida     |
| ...       | ...  | (truncated) |

### Example `survey` sheet

| type             | name  | label          | image      | appearance |
|------------------|-------|----------------|------------|------------|
| select_one state | state | Choose a state | US_MAP.svg | image-map  |

1. In the `choices` sheet of your XLSForm, make a list with a row for each `<path>` element in your SVG file. The values in the `name` column (e.g. `AK`, `HI`) of your `choices` sheet must correspond to the `id` attributes of your SVG `<path>` elements;
1. In the `survey` sheet, add a row for a [`select_one` question](http://xlsform.org/#multiple-choice) referencing the `list name` you used in the `choices` sheet (`state` in the example above);
1. Write the name of your SVG map file (e.g. `US_MAP.svg`) to the `image` column for this question;
1. Input the value `image-map` into the `appearance` column for this question;
1. Add any other desired questions to your XLSForm; save, upload, and deploy it to KoBoToolbox;
1. Access the settings for this deployed project and click the **+Add Document** button under the **Existing Form Files** heading;
1. Upload your SVG map file, making sure that the name of the file matches the name used in the `image` column of the `survey` sheet.

## Applying custom styling to the SVG map

You may apply custom styling to the SVG map, e.g. giving certain regions a particular fill color. The instructions below assume you have already followed the above procedures for creating a SVG map file and XLSForm.

### Example `survey` sheet with custom styling

| type | name | label | image | appearance | body::kb:image-customization | calculation |
| - | - | - | - | - | - | - |
| select_one state | state | Choose a state | US_MAP.svg | image-map image-customization | ${state_style} |  |
| calculate | state_style |  |  |  |  | concat('{"CA":{"fill":"cyan"},"selected": { "stroke": "magenta", "stroke-width": "10"}}') |

1. Add a [`settings` sheet](http://xlsform.org/#settings) to your XLSForm if it does not have one already;
1. Add a new column called `namespaces` with the value `kb="http://kobotoolbox.org/xforms"` in the cell below;
1. Add two new columns to your `survey` sheet:
    * `body::kb:image-customization`;
    * `calculation`;
1. Add a row for a new `calculate` question. This calculation will construct a JSON representation of SVG style instructions that will be applied to your map:
    1. In the `calculation` column, enter an expression that evaluates to stringified JSON in the format `{"id of SVG <path> element": {"style attribute name": "style attribute value"}}`;
    1. If you want to use a static string, enclose it in a `concat()` call, e.g. `concat('{"CA":{"fill":"cyan"}}')`;
    1. To control the appearance of a selected region, use the `selected` property in place of a SVG `<path>` element's `id`, e.g. `{"selected": { "stroke": "magenta", "stroke-width": "10"}}`;
    1. Create a new `calculate` question for each SVG map that you want to style individually;
1. For each row with a `select_one image-map` question:
    1. Append `image-customization` to the `apperance` column, so that the full value reads `image-map image-customization`;
    1. In the `body::kb:image-customization` column, enter a reference to a _calculate_ question, e.g. `${state_style}`. You [cannot](https://github.com/XLSForm/pyxform/issues/97) enter the JSON style instructions here directly;
1. Upload your modified XLSForm and redeploy.

## Complex example with repeating group and calculated styling ([download XLSForm](https://github.com/kobotoolbox/wfp-svg-map-documentation/blob/master/Example%20XLSForm.xlsx))

This example uses the [US_MAP.svg](https://github.com/kobotoolbox/wfp-svg-map-documentation/blob/master/US_MAP.svg?short_path=ab2ce7a) map file. It contains a repeating group of two questions:

1. Which state was affected?
1. What level of destruction was experienced?

Question (1) displays a clickable SVG map. Question (2) is a standard `select_one` question, but the respondent's choice causes the fill color of the state chosen in question (1) to change. All repeated instances of question (1) share the same styling, so if a respondent chose `AK` in the first instance and `CA` in the second, both maps show fill color for both `AK` and `CA`. This facilitates quickly referencing previous responses when entering new ones.

### Example `settings` sheet

The `namespaces` setting is required for any form with custom SVG styling.

| namespaces                         |
| ---------------------------------- |
| kb="http://kobotoolbox.org/xforms" |

### Example `choices` sheet

Note that the `destruction` list is a standard choice list that does not correspond to anything in the SVG file.

| list name   | name | label                |
|-------------|------|----------------------|
| destruction | a    | no or limited impact |
| destruction | b    | moderate impact      |
| destruction | c    | high impact          |
| destruction | d    | severe impact        |
| state       | AK   | Alaska               |
| state       | HI   | Hawaii               |
| state       | AL   | Alabama              |
| state       | AR   | Arkansas             |
| state       | AZ   | Arizona              |
| state       | CA   | California           |
| state       | CO   | Colorado             |
| state       | CT   | Connecticut          |
| state       | DE   | Delaware             |
| state       | FL   | Florida              |
| ...         | ...  | (truncated)          |

### Example `survey` sheet

| type | name | label | image | appearance | body::kb:image-customization | calculation |
| - | - | - | - | - | - | - |
| begin repeat |  |  |  |  |  |  |
| select_one state | state | Which state was affected? | US_MAP.svg | image-map image-customization | ${state_combined_style} |  |
| select_one destruction | destruction | What level of destruction was experienced? |  |  |  |  |
| calculate | state_color |  |  |  |  | if(${destruction} = 'a', 'green', (if(${destruction} = 'b', 'yellow', (if(${destruction} = 'c', 'orange', (if(${destruction} = 'd', 'red', 'inherit'))))))) |
| calculate | state_instance_style |  |  |  |  | if (${state} != '' and ${destruction} != '', concat('"', ${state}, '": {"fill":"', ${state_color}, '"},'), '') |
| end repeat |  |  |  |  |  |  |
| calculate | state_combined_style |  |  |  |  | concat('{', ${state_instance_style}, '"selected": { "stroke": "yellow", "stroke-width": "10"}', '}') |

### Explanation of calculations

* `state_color`: calculate a [SVG color name](http://www.december.com/html/spec/colorsvg.html) based on the level of `destruction` chosen.

  ```
  if(${destruction} = 'a', 'green', (
    if(${destruction} = 'b', 'yellow', (
      if(${destruction} = 'c', 'orange', (
        if(${destruction} = 'd', 'red', 'inherit')
      ))
    ))
  ))
  ```

* `state_instance_style`: when selections have been made for both `state` and `destruction`, construct **partial** JSON style instructions in the format of the following example: `"AK": {"fill": "orange"},`. Note carefully that this is partial JSON because it has no leading and trailing braces&mdash;this is required for subsequent concatenation in `state_combined_style`.

  ```
  if (${state} != '' and ${destruction} != '',
      concat('"', ${state}, '": {"fill": "', ${state_color}, '"},'),
  '')
  ```


* `state_combined_style`: concatenate all the `state_instance_style`s together, adding a yellow stroke around the currently selected `state`, and enclose the result in braces to render valid JSON.

  ```
  concat('{', ${state_instance_style}, '"selected": { "stroke": "yellow", "stroke-width": "10"}', '}')
  ```
  
  Note: adding a `selected` style is not required, but the trailing comma in `state_instance_style` results in invalid JSON unless at least one property follows `${state_instance_style}`. This could be `"": null` if no additional styles are desired.
