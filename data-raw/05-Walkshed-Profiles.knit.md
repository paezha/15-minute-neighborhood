
<!-- rnb-text-begin -->

---
title: "Profiling Walksheds"
author: "Antonio Paez"
date: "2024-04-23"
output: html_notebook
---


<!-- rnb-text-end -->



<!-- rnb-text-begin -->


In this document we profile the ped sheds based on their network attributes.

# Preliminaries

Load packages:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGlicmFyeShicm9vbSkgIyBDb252ZXJ0IFN0YXRpc3RpY2FsIE9iamVjdHMgaW50byBUaWR5IFRpYmJsZXNcbmxpYnJhcnkoZHBseXIpICMgQSBHcmFtbWFyIG9mIERhdGEgTWFuaXB1bGF0aW9uXG5saWJyYXJ5KGV2dHJlZSkgIyBFdm9sdXRpb25hcnkgTGVhcm5pbmcgb2YgR2xvYmFsbHkgT3B0aW1hbCBUcmVlc1xubGlicmFyeShnZ3BhcnR5KSAjICdnZ3Bsb3QnIFZpc3VhbGl6YXRpb25zIGZvciB0aGUgJ3BhcnR5a2l0JyBQYWNrYWdlXG5saWJyYXJ5KGdncGxvdDIpICMgQ3JlYXRlIEVsZWdhbnQgRGF0YSBWaXN1YWxpc2F0aW9ucyBVc2luZyB0aGUgR3JhbW1hciBvZiBHcmFwaGljc1xubGlicmFyeShnbHVlKSAjIEludGVycHJldGVkIFN0cmluZyBMaXRlcmFsc1xubGlicmFyeShndCkgIyBFYXNpbHkgQ3JlYXRlIFByZXNlbnRhdGlvbi1SZWFkeSBEaXNwbGF5IFRhYmxlc1xubGlicmFyeShwYXRjaHdvcmspICMgVGhlIENvbXBvc2VyIG9mIFBsb3RzXG5saWJyYXJ5KHRpZHlyKSAjIFRpZHkgTWVzc3kgRGF0YVxubGlicmFyeSh0cmVlKSAjIENsYXNzaWZpY2F0aW9uIGFuZCBSZWdyZXNzaW9uIFRyZWVzXG5saWJyYXJ5KHNmKSAjIFNpbXBsZSBGZWF0dXJlcyBmb3IgUlxubGlicmFyeShza2ltcikgIyBDb21wYWN0IGFuZCBGbGV4aWJsZSBTdW1tYXJpZXMgb2YgRGF0YVxubGlicmFyeShTT01icmVybykgIyBTT00gQm91bmQgdG8gUmVhbGl6ZSBFdWNsaWRlYW4gYW5kIFJlbGF0aW9uYWwgT3V0cHV0c1xubGlicmFyeShzdHJpbmdyKSAjIFNpbXBsZSwgQ29uc2lzdGVudCBXcmFwcGVycyBmb3IgQ29tbW9uIFN0cmluZyBPcGVyYXRpb25zXG5saWJyYXJ5KHVuaXRzKSAjIE1lYXN1cmVtZW50IFVuaXRzIGZvciBSIFZlY3RvcnNcbmxpYnJhcnkodmFjY0hhbWlsdG9uKSAjIEEgRGF0YSBQYWNrYWdlIHRvIEVzdGltYXRlIEFjY2Vzc2liaWxpdHkgb2YgVmFjY2luYXRpb24gU2l0ZXMgaW4gSGFtaWx0b24sIE9OXG5gYGAifQ== -->

```r
library(broom) # Convert Statistical Objects into Tidy Tibbles
library(dplyr) # A Grammar of Data Manipulation
library(evtree) # Evolutionary Learning of Globally Optimal Trees
library(ggparty) # 'ggplot' Visualizations for the 'partykit' Package
library(ggplot2) # Create Elegant Data Visualisations Using the Grammar of Graphics
library(glue) # Interpreted String Literals
library(gt) # Easily Create Presentation-Ready Display Tables
library(patchwork) # The Composer of Plots
library(tidyr) # Tidy Messy Data
library(tree) # Classification and Regression Trees
library(sf) # Simple Features for R
library(skimr) # Compact and Flexible Summaries of Data
library(SOMbrero) # SOM Bound to Realize Euclidean and Relational Outputs
library(stringr) # Simple, Consistent Wrappers for Common String Operations
library(units) # Measurement Units for R Vectors
library(vaccHamilton) # A Data Package to Estimate Accessibility of Vaccination Sites in Hamilton, ON
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Load data:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubG9hZChmaWxlID0gXCJhbWVuaXRpZXNfd2Fsa3NoZWRfZGEucmRhXCIpXG5sb2FkKGZpbGUgPSBcImhhbWlsdG9uX25ldC5yZGFcIilcbmBgYCJ9 -->

```r
load(file = "amenities_walkshed_da.rda")
load(file = "hamilton_net.rda")
```

<!-- rnb-source-end -->

<!-- rnb-output-begin eyJkYXRhIjoiV2FybmluZzogY2Fubm90IG9wZW4gY29tcHJlc3NlZCBmaWxlICdoYW1pbHRvbl9uZXQucmRhJywgcHJvYmFibGUgcmVhc29uICdObyBzdWNoIGZpbGUgb3IgZGlyZWN0b3J5J0Vycm9yIGluIHJlYWRDaGFyKGNvbiwgNUwsIHVzZUJ5dGVzID0gVFJVRSkgOiBjYW5ub3Qgb3BlbiB0aGUgY29ubmVjdGlvblxuIn0= -->

```
Warning: cannot open compressed file 'hamilton_net.rda', probable reason 'No such file or directory'Error in readChar(con, 5L, useBytes = TRUE) : cannot open the connection
```



<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


## Classification tree

The aim is to train a classification tree to profile urban and suburban walksheds based on the attributes of the network. We need to drop 1 observation that has an NA in `transitivity`:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZCA8LSB3YWxrc2hlZHNfZGFfbmV0X3ZhcnMgfD5cbiAgICAgICAgICAgICAgICAgIGRyb3BfbmEoKSB8PlxuICBtdXRhdGUobm9ybWFsaXplZF9tb3RpZnNfMyA9IG1vdGlmc18zL25fZWRnZXMsXG4gICAgICAgICBub3JtYWxpemVkX21vdGlmc180ID0gbW90aWZzXzQvbl9lZGdlcylcbmBgYCJ9 -->

```r
d <- walksheds_da_net_vars |>
                  drop_na() |>
  mutate(normalized_motifs_3 = motifs_3/n_edges,
         normalized_motifs_4 = motifs_4/n_edges)
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



We use a technique with evolutionary trees - an alternative to classification trees that is less affecte by the greediness of the algorithm. Since evolutionary trees have some randomness (for the initial values), we can try several to find whether there are patterns. In this case we try four different random seed. 

Tree 1 with seed 1 (912571):

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Tree 2 with seed 2 (94766):

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Tree 3 with seed 3 (46125):

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Tree 4 with seed 4 (556090)

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Create a function to estimate the missclassification rate and the evaluation function:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


All trees have similar fit.

Plot the trees:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Notice that edge_density and normalized_motifs_3 are the dominant attributes in the classification trees. Trees 1, 3, and 4 have the same general structure, and in fact trees 3 and 4 are identical. Based on this we choose to use `tree3`.


Check the break labels of `tree3` to round:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->




<!-- rnb-text-end -->



<!-- rnb-text-begin -->



<!-- rnb-text-end -->



<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


```{=html}

<!-- rnb-html-begin eyJtZXRhZGF0YSI6eyJjbGFzc2VzIjpbInNoaW55LnRhZyJdLCJzaXppbmdQb2xpY3kiOltdfX0= -->

<div id="naxpbldilw" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
  <style>#naxpbldilw table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#naxpbldilw thead, #naxpbldilw tbody, #naxpbldilw tfoot, #naxpbldilw tr, #naxpbldilw td, #naxpbldilw th {
  border-style: none;
}

#naxpbldilw p {
  margin: 0;
  padding: 0;
}

#naxpbldilw .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 30px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#naxpbldilw .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#naxpbldilw .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#naxpbldilw .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#naxpbldilw .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#naxpbldilw .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#naxpbldilw .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#naxpbldilw .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#naxpbldilw .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#naxpbldilw .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#naxpbldilw .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#naxpbldilw .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#naxpbldilw .gt_spanner_row {
  border-bottom-style: hidden;
}

#naxpbldilw .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#naxpbldilw .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#naxpbldilw .gt_from_md > :first-child {
  margin-top: 0;
}

#naxpbldilw .gt_from_md > :last-child {
  margin-bottom: 0;
}

#naxpbldilw .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#naxpbldilw .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#naxpbldilw .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#naxpbldilw .gt_row_group_first td {
  border-top-width: 2px;
}

#naxpbldilw .gt_row_group_first th {
  border-top-width: 2px;
}

#naxpbldilw .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#naxpbldilw .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#naxpbldilw .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#naxpbldilw .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#naxpbldilw .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#naxpbldilw .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#naxpbldilw .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#naxpbldilw .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#naxpbldilw .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#naxpbldilw .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#naxpbldilw .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#naxpbldilw .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#naxpbldilw .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#naxpbldilw .gt_left {
  text-align: left;
}

#naxpbldilw .gt_center {
  text-align: center;
}

#naxpbldilw .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#naxpbldilw .gt_font_normal {
  font-weight: normal;
}

#naxpbldilw .gt_font_bold {
  font-weight: bold;
}

#naxpbldilw .gt_font_italic {
  font-style: italic;
}

#naxpbldilw .gt_super {
  font-size: 65%;
}

#naxpbldilw .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#naxpbldilw .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#naxpbldilw .gt_indent_1 {
  text-indent: 5px;
}

#naxpbldilw .gt_indent_2 {
  text-indent: 10px;
}

#naxpbldilw .gt_indent_3 {
  text-indent: 15px;
}

#naxpbldilw .gt_indent_4 {
  text-indent: 20px;
}

#naxpbldilw .gt_indent_5 {
  text-indent: 25px;
}
</style>
  <table class="gt_table" style="table-layout: fixed;; width: 0px" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <colgroup>
    <col style="width:100px;"/>
    <col style="width:100px;"/>
    <col style="width:100px;"/>
    <col style="width:100px;"/>
    <col style="width:100px;"/>
    <col style="width:100px;"/>
    <col style="width:100px;"/>
    <col style="width:100px;"/>
    <col style="width:100px;"/>
    <col style="width:100px;"/>
  </colgroup>
  <thead>
    
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" scope="col" id="leaf">leaf</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col" id="Type">Type</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="mean">mean</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="sd">sd</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="p0">p0</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="p25">p25</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="p50">p50</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="p75">p75</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="p100">p100</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col" id="hist">hist</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td headers="leaf" class="gt_row gt_center">Leaf 1</td>
<td headers="Type" class="gt_row gt_left">Typically suburban</td>
<td headers="mean" class="gt_row gt_right">1.22</td>
<td headers="sd" class="gt_row gt_right">0.17</td>
<td headers="p0" class="gt_row gt_right">0.79</td>
<td headers="p25" class="gt_row gt_right">1.13</td>
<td headers="p50" class="gt_row gt_right">1.25</td>
<td headers="p75" class="gt_row gt_right">1.32</td>
<td headers="p100" class="gt_row gt_right">1.50</td>
<td headers="hist" class="gt_row gt_left">▁▃▅▇▃</td></tr>
    <tr><td headers="leaf" class="gt_row gt_center">Leaf 2</td>
<td headers="Type" class="gt_row gt_left">Typically urban</td>
<td headers="mean" class="gt_row gt_right">1.45</td>
<td headers="sd" class="gt_row gt_right">0.20</td>
<td headers="p0" class="gt_row gt_right">0.72</td>
<td headers="p25" class="gt_row gt_right">1.33</td>
<td headers="p50" class="gt_row gt_right">1.50</td>
<td headers="p75" class="gt_row gt_right">1.60</td>
<td headers="p100" class="gt_row gt_right">1.86</td>
<td headers="hist" class="gt_row gt_left">▁▂▃▇▂</td></tr>
    <tr><td headers="leaf" class="gt_row gt_center">Leaf 3</td>
<td headers="Type" class="gt_row gt_left">Typically suburban</td>
<td headers="mean" class="gt_row gt_right">1.02</td>
<td headers="sd" class="gt_row gt_right">0.28</td>
<td headers="p0" class="gt_row gt_right">0.14</td>
<td headers="p25" class="gt_row gt_right">0.84</td>
<td headers="p50" class="gt_row gt_right">1.04</td>
<td headers="p75" class="gt_row gt_right">1.21</td>
<td headers="p100" class="gt_row gt_right">1.67</td>
<td headers="hist" class="gt_row gt_left">▁▃▇▇▂</td></tr>
    <tr><td headers="leaf" class="gt_row gt_center">Leaf 4</td>
<td headers="Type" class="gt_row gt_left">Typically urban</td>
<td headers="mean" class="gt_row gt_right">1.35</td>
<td headers="sd" class="gt_row gt_right">0.35</td>
<td headers="p0" class="gt_row gt_right">0.59</td>
<td headers="p25" class="gt_row gt_right">1.07</td>
<td headers="p50" class="gt_row gt_right">1.50</td>
<td headers="p75" class="gt_row gt_right">1.64</td>
<td headers="p100" class="gt_row gt_right">1.76</td>
<td headers="hist" class="gt_row gt_left">▂▃▃▃▇</td></tr>
  </tbody>
  
  
</table>
</div>


<!-- rnb-html-end -->

```

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



Typically suburban networks contain many unique patterns (few motifs).

0.00591859212548868 & normalized_motifs_3 < 0.86734693877551 ~ "Leaf 1",
                          edge_density < 0.00591859212548868 & normalized_motifs_3 >= 0.86734693877551 ~ "Leaf 2",
                          edge_density >= 0.00591859212548868 & normalized_motifs_3 < 0.967741935483871 ~ "Leaf 3",


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc2V0LnNlZWQoMjk5MTkpXG5cbmxlYWZfMV9wbG90IDwtIHN0X2ludGVyc2VjdGlvbihoYW1pbHRvbl9uZXQkZWRnZXMsXG4gICAgICAgICAgICAgd2Fsa3NoZWRzX2RhIHw+IFxuICBmaWx0ZXIoR2VvVUlEID09ICh3YWxrc2hlZHNfZGFfbmV0X3ZhcnMgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgbXV0YXRlKG5vcm1hbGl6ZWRfbW90aWZzXzMgPSBtb3RpZnNfMy9uX2VkZ2VzKSB8PlxuICAgICAgICAgICAgICAgICAgICAgIGZpbHRlcihlZGdlX2RlbnNpdHkgPCAwLjAwNTkxODU5MjEyNTQ4ODY4LFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICBub3JtYWxpemVkX21vdGlmc18zIDwgMC44NjczNDY5Mzg3NzU1MSwgXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgIFR5cGUgPT0gXCJTdWJ1cmJhblwiKSB8PiBcbiAgICAgICAgICAgICAgICAgICAgICBzbGljZV9zYW1wbGUobj0xKSB8PiBcbiAgICAgICAgICAgICAgICAgICAgICBwdWxsKEdlb1VJRCkpKSlcbmBgYCJ9 -->

```r
set.seed(29919)

leaf_1_plot <- st_intersection(hamilton_net$edges,
             walksheds_da |> 
  filter(GeoUID == (walksheds_da_net_vars |> 
                      mutate(normalized_motifs_3 = motifs_3/n_edges) |>
                      filter(edge_density < 0.00591859212548868,
                             normalized_motifs_3 < 0.86734693877551, 
                             Type == "Suburban") |> 
                      slice_sample(n=1) |> 
                      pull(GeoUID))))
```

<!-- rnb-source-end -->

<!-- rnb-output-begin eyJkYXRhIjoiV2FybmluZzogYXR0cmlidXRlIHZhcmlhYmxlcyBhcmUgYXNzdW1lZCB0byBiZSBzcGF0aWFsbHkgY29uc3RhbnQgdGhyb3VnaG91dCBhbGwgZ2VvbWV0cmllc1xuIn0= -->

```
Warning: attribute variables are assumed to be spatially constant throughout all geometries
```



<!-- rnb-output-end -->

<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGVhZl8xX3Bsb3QgPC0gaGFtaWx0b25fbmV0JGVkZ2VzIHw+XG4gIGZpbHRlcihlZGdlX2luZGV4ICVpbiUgbGVhZl8xX3Bsb3QkZWRnZV9pbmRleClcblxuI2xlYWZfMV9wbG90IDwtIFxuICBsZWFmXzFfcGxvdCB8PlxuICBnZ3Bsb3QoKSArXG4gIGdlb21fc2YoKSArXG4gIGdndGl0bGUoXCJlZGdlIGRlbnNpdHkgPCAwLjAwNiwgbW90aWZzIDwgMC44NjdcIikgK1xuICB0aGVtZV92b2lkKClcbmBgYCJ9 -->

```r
leaf_1_plot <- hamilton_net$edges |>
  filter(edge_index %in% leaf_1_plot$edge_index)

#leaf_1_plot <- 
  leaf_1_plot |>
  ggplot() +
  geom_sf() +
  ggtitle("edge density < 0.006, motifs < 0.867") +
  theme_void()
```

<!-- rnb-source-end -->

<!-- rnb-plot-begin -->

<img src="data:image/png;base64,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" />

<!-- rnb-plot-end -->

<!-- rnb-plot-begin eyJoZWlnaHQiOjQzMi42MzI5LCJ3aWR0aCI6NzAwLCJzaXplX2JlaGF2aW9yIjowLCJjb25kaXRpb25zIjpbXX0= -->

<img src="data:image/png;base64,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" />

<!-- rnb-plot-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc2V0LnNlZWQoODQ0NTEpXG5cbmxlYWZfMl9wbG90IDwtIHN0X2ludGVyc2VjdGlvbihoYW1pbHRvbl9uZXQkZWRnZXMsXG4gICAgICAgICAgICAgd2Fsa3NoZWRzX2RhIHw+IFxuICBmaWx0ZXIoR2VvVUlEID09ICh3YWxrc2hlZHNfZGFfbmV0X3ZhcnMgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgbXV0YXRlKG5vcm1hbGl6ZWRfbW90aWZzXzMgPSBtb3RpZnNfMy9uX2VkZ2VzKSB8PlxuICAgICAgICAgICAgICAgICAgICAgIGZpbHRlcihlZGdlX2RlbnNpdHkgPj0gMC4wMDU5MTg1OTIxMjU0ODg2OCxcbiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbm9ybWFsaXplZF9tb3RpZnNfMyA8IDAuOTY3NzQxOTM1NDgzODcxLCBcbiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgVHlwZSA9PSBcIlN1YnVyYmFuXCIpIHw+IFxuICAgICAgICAgICAgICAgICAgICAgIHNsaWNlX3NhbXBsZShuPTEpIHw+IFxuICAgICAgICAgICAgICAgICAgICAgIHB1bGwoR2VvVUlEKSkpKVxuYGBgIn0= -->

```r
set.seed(84451)

leaf_2_plot <- st_intersection(hamilton_net$edges,
             walksheds_da |> 
  filter(GeoUID == (walksheds_da_net_vars |> 
                      mutate(normalized_motifs_3 = motifs_3/n_edges) |>
                      filter(edge_density >= 0.00591859212548868,
                             normalized_motifs_3 < 0.967741935483871, 
                             Type == "Suburban") |> 
                      slice_sample(n=1) |> 
                      pull(GeoUID))))
```

<!-- rnb-source-end -->

<!-- rnb-output-begin eyJkYXRhIjoiV2FybmluZzogYXR0cmlidXRlIHZhcmlhYmxlcyBhcmUgYXNzdW1lZCB0byBiZSBzcGF0aWFsbHkgY29uc3RhbnQgdGhyb3VnaG91dCBhbGwgZ2VvbWV0cmllc1xuIn0= -->

```
Warning: attribute variables are assumed to be spatially constant throughout all geometries
```



<!-- rnb-output-end -->

<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGVhZl8yX3Bsb3QgPC0gaGFtaWx0b25fbmV0JGVkZ2VzIHw+XG4gIGZpbHRlcihlZGdlX2luZGV4ICVpbiUgbGVhZl8yX3Bsb3QkZWRnZV9pbmRleClcblxubGVhZl8yX3Bsb3QgfD5cbiAgZ2dwbG90KCkgK1xuICBnZW9tX3NmKCkgK1xuICBnZ3RpdGxlKFwiZWRnZSBkZW5zaXR5IDwgMC4wMDYsIG1vdGlmcyA+PSAwLjk2OFwiKSArXG4gIHRoZW1lX3ZvaWQoKVxuYGBgIn0= -->

```r
leaf_2_plot <- hamilton_net$edges |>
  filter(edge_index %in% leaf_2_plot$edge_index)

leaf_2_plot |>
  ggplot() +
  geom_sf() +
  ggtitle("edge density < 0.006, motifs >= 0.968") +
  theme_void()
```

<!-- rnb-source-end -->

<!-- rnb-plot-begin -->

<img src="data:image/png;base64,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" />

<!-- rnb-plot-end -->

<!-- rnb-plot-begin eyJoZWlnaHQiOjQzMi42MzI5LCJ3aWR0aCI6NzAwLCJzaXplX2JlaGF2aW9yIjowLCJjb25kaXRpb25zIjpbXX0= -->

<img src="data:image/png;base64,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" />

<!-- rnb-plot-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



Typically urban networks contain isomorphic patterns that repeat (large number of motifs).


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc2V0LnNlZWQoOTIzNDgpXG5cbmxlYWZfMV9wbG90IDwtIHN0X2ludGVyc2VjdGlvbihoYW1pbHRvbl9uZXQkZWRnZXMsXG4gICAgICAgICAgICAgd2Fsa3NoZWRzX2RhIHw+IFxuICBmaWx0ZXIoR2VvVUlEID09ICh3YWxrc2hlZHNfZGFfbmV0X3ZhcnMgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgbXV0YXRlKG5vcm1hbGl6ZWRfbW90aWZzXzMgPSBtb3RpZnNfMy9uX2VkZ2VzKSB8PlxuICAgICAgICAgICAgICAgICAgICAgIGZpbHRlcihlZGdlX2RlbnNpdHkgPCAwLjAwNTkxODU5MjEyNTQ4ODY4LFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICBub3JtYWxpemVkX21vdGlmc18zID49IDAuODY3MzQ2OTM4Nzc1NTEsIFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICBUeXBlID09IFwiVXJiYW5cIikgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgc2xpY2Vfc2FtcGxlKG49MSkgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgcHVsbChHZW9VSUQpKSkpXG5gYGAifQ== -->

```r
set.seed(92348)

leaf_1_plot <- st_intersection(hamilton_net$edges,
             walksheds_da |> 
  filter(GeoUID == (walksheds_da_net_vars |> 
                      mutate(normalized_motifs_3 = motifs_3/n_edges) |>
                      filter(edge_density < 0.00591859212548868,
                             normalized_motifs_3 >= 0.86734693877551, 
                             Type == "Urban") |> 
                      slice_sample(n=1) |> 
                      pull(GeoUID))))
```

<!-- rnb-source-end -->

<!-- rnb-output-begin eyJkYXRhIjoiV2FybmluZzogYXR0cmlidXRlIHZhcmlhYmxlcyBhcmUgYXNzdW1lZCB0byBiZSBzcGF0aWFsbHkgY29uc3RhbnQgdGhyb3VnaG91dCBhbGwgZ2VvbWV0cmllc1xuIn0= -->

```
Warning: attribute variables are assumed to be spatially constant throughout all geometries
```



<!-- rnb-output-end -->

<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGVhZl8xX3Bsb3QgPC0gaGFtaWx0b25fbmV0JGVkZ2VzIHw+XG4gIGZpbHRlcihlZGdlX2luZGV4ICVpbiUgbGVhZl8xX3Bsb3QkZWRnZV9pbmRleClcblxuI2xlYWZfMV9wbG90IDwtIFxuICBsZWFmXzFfcGxvdCB8PlxuICBnZ3Bsb3QoKSArXG4gIGdlb21fc2YoKSArXG4gIGdndGl0bGUoXCJlZGdlIGRlbnNpdHkgPCAwLjAwNiwgbW90aWZzID49IDAuODY3XCIpICtcbiAgdGhlbWVfdm9pZCgpXG5gYGAifQ== -->

```r
leaf_1_plot <- hamilton_net$edges |>
  filter(edge_index %in% leaf_1_plot$edge_index)

#leaf_1_plot <- 
  leaf_1_plot |>
  ggplot() +
  geom_sf() +
  ggtitle("edge density < 0.006, motifs >= 0.867") +
  theme_void()
```

<!-- rnb-source-end -->

<!-- rnb-plot-begin -->

<img src="data:image/png;base64,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" />

<!-- rnb-plot-end -->

<!-- rnb-plot-begin eyJoZWlnaHQiOjQzMi42MzI5LCJ3aWR0aCI6NzAwLCJzaXplX2JlaGF2aW9yIjowLCJjb25kaXRpb25zIjpbXX0= -->

<img src="data:image/png;base64,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" />

<!-- rnb-plot-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc2V0LnNlZWQoMzkyODQ3KVxuXG5sZWFmXzRfcGxvdCA8LSBzdF9pbnRlcnNlY3Rpb24oaGFtaWx0b25fbmV0JGVkZ2VzLFxuICAgICAgICAgICAgIHdhbGtzaGVkc19kYSB8PiBcbiAgZmlsdGVyKEdlb1VJRCA9PSAod2Fsa3NoZWRzX2RhX25ldF92YXJzIHw+IFxuICAgICAgICAgICAgICAgICAgICAgIG11dGF0ZShub3JtYWxpemVkX21vdGlmc18zID0gbW90aWZzXzMvbl9lZGdlcykgfD5cbiAgICAgICAgICAgICAgICAgICAgICBmaWx0ZXIoZWRnZV9kZW5zaXR5ID49IDAuMDA1OTE4NTkyMTI1NDg4NjgsXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgIG5vcm1hbGl6ZWRfbW90aWZzXzMgPj0gMC45Njc3NDE5MzU0ODM4NzEsIFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICBUeXBlID09IFwiVXJiYW5cIikgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgc2xpY2Vfc2FtcGxlKG49MSkgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgcHVsbChHZW9VSUQpKSkpXG5gYGAifQ== -->

```r
set.seed(392847)

leaf_4_plot <- st_intersection(hamilton_net$edges,
             walksheds_da |> 
  filter(GeoUID == (walksheds_da_net_vars |> 
                      mutate(normalized_motifs_3 = motifs_3/n_edges) |>
                      filter(edge_density >= 0.00591859212548868,
                             normalized_motifs_3 >= 0.967741935483871, 
                             Type == "Urban") |> 
                      slice_sample(n=1) |> 
                      pull(GeoUID))))
```

<!-- rnb-source-end -->

<!-- rnb-output-begin eyJkYXRhIjoiV2FybmluZzogYXR0cmlidXRlIHZhcmlhYmxlcyBhcmUgYXNzdW1lZCB0byBiZSBzcGF0aWFsbHkgY29uc3RhbnQgdGhyb3VnaG91dCBhbGwgZ2VvbWV0cmllc1xuIn0= -->

```
Warning: attribute variables are assumed to be spatially constant throughout all geometries
```



<!-- rnb-output-end -->

<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGVhZl80X3Bsb3QgPC0gaGFtaWx0b25fbmV0JGVkZ2VzIHw+XG4gIGZpbHRlcihlZGdlX2luZGV4ICVpbiUgbGVhZl80X3Bsb3QkZWRnZV9pbmRleClcblxubGVhZl80X3Bsb3QgfD5cbiAgZ2dwbG90KCkgK1xuICBnZW9tX3NmKCkgK1xuICBnZ3RpdGxlKFwiZWRnZSBkZW5zaXR5IDwgMC4wMDYsIG1vdGlmcyA+PSAwLjk2OFwiKSArXG4gIHRoZW1lX3ZvaWQoKVxuYGBgIn0= -->

```r
leaf_4_plot <- hamilton_net$edges |>
  filter(edge_index %in% leaf_4_plot$edge_index)

leaf_4_plot |>
  ggplot() +
  geom_sf() +
  ggtitle("edge density < 0.006, motifs >= 0.968") +
  theme_void()
```

<!-- rnb-source-end -->

<!-- rnb-plot-begin -->

<img src="data:image/png;base64,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" />

<!-- rnb-plot-end -->

<!-- rnb-plot-begin eyJoZWlnaHQiOjQzMi42MzI5LCJ3aWR0aCI6NzAwLCJzaXplX2JlaGF2aW9yIjowLCJjb25kaXRpb25zIjpbXX0= -->

<img src="data:image/png;base64,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" />

<!-- rnb-plot-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



## Analysis of pedshed areas as a function of network attributes


<!-- rnb-text-end -->



<!-- rnb-text-begin -->



<!-- rnb-text-end -->



<!-- rnb-text-begin -->



<!-- rnb-text-end -->



<!-- rnb-text-begin -->



<!-- rnb-text-end -->



<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->



<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!--
Check the break labels of `tree3` to round:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZ2dwYXJ0eSh0cmVlMykkZGF0YSRicmVha3NfbGFiZWxcbmBgYCJ9 -->

```r
ggparty(tree3)$data$breaks_label
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Plot the tree with the rounded break labels:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZ2dwYXJ0eSh0cmVlMykgK1xuICBnZW9tX2VkZ2UoKSArXG4gIGdlb21fZWRnZV9sYWJlbChhZXMobGFiZWwgPSBzdHJfcmVwbGFjZShicmVha3NfbGFiZWwsIFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgXCIwLjAyMDczMTE2MzYyMDE1MTZcIiwgXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBcIjAuMDIxXCIpKSkgK1xuICBnZW9tX25vZGVfc3BsaXR2YXIoYWVzKGxhYmVsID0gaWZlbHNlKGtpZHMgPT0gMCxcbiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBnbHVlKFwiIG4gPSB7bm9kZXNpemV9XCIpLCBcbiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBzcGxpdHZhcikpLFxuICAgICAgICAgICAgICAgICAgICAgbnVkZ2VfeCA9IDAuMCxcbiAgICAgICAgICAgICAgICAgICAgIGlkcyA9IGMoMSwgMiwgNSkpICtcbiAgZ2VvbV9ub2RlX3NwbGl0dmFyKGFlcyhsYWJlbCA9IGlmZWxzZShraWRzID09IDAsXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgZ2x1ZShcIiBuID0ge25vZGVzaXplfVwiKSwgXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgc3BsaXR2YXIpKSxcbiAgICAgICAgICAgICAgICAgICAgIG51ZGdlX3ggPSAwLjAyLFxuICAgICAgICAgICAgICAgICAgICAgaWRzID0gYygzLCA0LCA2LCA3KSkgK1xuICBnZW9tX25vZGVfcGxvdChnZ2xpc3QgPSBsaXN0KGdlb21fYmFyKGFlcyh4ID0gXCJcIiwgXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGZpbGwgPSBUeXBlKSxcbiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBwb3NpdGlvbiA9IHBvc2l0aW9uX2ZpbGwoKSksXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgeGxhYihcIlR5cGVcIiksXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgdGhlbWVfbWluaW1hbCgpKSxcbiAgICAgICAgICAgICAgICAgIyBkcmF3IG9ubHkgb25lIGxhYmVsIGZvciBlYWNoIGF4aXNcbiAgICAgICAgICAgICAgICAgc2hhcmVkX2F4aXNfbGFiZWxzID0gVFJVRSxcbiAgICAgICAgICAgICAgICAgIyBkcmF3IGxpbmUgYmV0d2VlbiB0cmVlIGFuZCBsZWdlbmRcbiAgICAgICAgICAgICAgICAgbGVnZW5kX3NlcGFyYXRvciA9IEZBTFNFXG4gICAgICAgICAgICAgICAgIClcbmBgYCJ9 -->

```r
ggparty(tree3) +
  geom_edge() +
  geom_edge_label(aes(label = str_replace(breaks_label, 
                                          "0.0207311636201516", 
                                          "0.021"))) +
  geom_node_splitvar(aes(label = ifelse(kids == 0,
                                        glue(" n = {nodesize}"), 
                                        splitvar)),
                     nudge_x = 0.0,
                     ids = c(1, 2, 5)) +
  geom_node_splitvar(aes(label = ifelse(kids == 0,
                                        glue(" n = {nodesize}"), 
                                        splitvar)),
                     nudge_x = 0.02,
                     ids = c(3, 4, 6, 7)) +
  geom_node_plot(gglist = list(geom_bar(aes(x = "", 
                                            fill = Type),
                                        position = position_fill()),
                               xlab("Type"),
                               theme_minimal()),
                 # draw only one label for each axis
                 shared_axis_labels = TRUE,
                 # draw line between tree and legend
                 legend_separator = FALSE
                 )
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Find examples of ped sheds in each of the leafs and plot the network for illustration.

Leaf 1:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuXG5zZWVkIDwtIHNhbXBsZS5pbnQoMTAwMDAwLCAxKVxuc2V0LnNlZWQoc2VlZClcbnNlZWRcbiM2OTcxN1xuXG5zdF9pbnRlcnNlY3Rpb24oaGFtaWx0b25fbmV0JGVkZ2VzLFxuICAgICAgICAgICAgIHdhbGtzaGVkcyB8PiBcbiAgZmlsdGVyKEdlb1VJRCA9PSAod2Fsa3NoZWRzX25ldF9hdHRyaWJ1dGVzIHw+IFxuICAgICAgICAgICAgICAgICAgICAgIGZpbHRlcih0cmFuc2l0aXZpdHkgPCAwLjAyMDczMTE2MzYyMDE1MTYsXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1vdGlmcyA8IDM3OCwgXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgIFR5cGUgPT0gXCJTdWJ1cmJhblwiKSB8PiBcbiAgICAgICAgICAgICAgICAgICAgICBzbGljZV9zYW1wbGUobj0xKSB8PiBcbiAgICAgICAgICAgICAgICAgICAgICBwdWxsKEdlb1VJRCkpKSkgfD5cbiAgZ2dwbG90KCkgK1xuICBnZW9tX3NmKClcbmBgYCJ9 -->

```r

seed <- sample.int(100000, 1)
set.seed(seed)
seed
#69717

st_intersection(hamilton_net$edges,
             walksheds |> 
  filter(GeoUID == (walksheds_net_attributes |> 
                      filter(transitivity < 0.0207311636201516,
                             motifs < 378, 
                             Type == "Suburban") |> 
                      slice_sample(n=1) |> 
                      pull(GeoUID)))) |>
  ggplot() +
  geom_sf()
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Leaf 2 (urban):

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc2VlZCA8LSBzYW1wbGUuaW50KDEwMDAwMCwgMSlcbnNldC5zZWVkKHNlZWQpXG5zZWVkXG4jODQ0NTEsIDQ0Nzg1XG5cbnN0X2ludGVyc2VjdGlvbihoYW1pbHRvbl9uZXQkZWRnZXMsXG4gICAgICAgICAgICAgd2Fsa3NoZWRzIHw+IFxuICBmaWx0ZXIoR2VvVUlEID09ICh3YWxrc2hlZHNfbmV0X2F0dHJpYnV0ZXMgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgZmlsdGVyKG1vdGlmcyA+PSAzNzgsIFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICB0cmFuc2l0aXZpdHkgPCAwLjAyMDczMTE2MzYyMDE1MTYsXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgIFR5cGUgPT0gXCJVcmJhblwiKSB8PiBcbiAgICAgICAgICAgICAgICAgICAgICBzbGljZV9zYW1wbGUobj0xKSB8PiBcbiAgICAgICAgICAgICAgICAgICAgICBwdWxsKEdlb1VJRCkpKSkgfD5cbiAgZ2dwbG90KCkgK1xuICBnZW9tX3NmKClcbmBgYCJ9 -->

```r
seed <- sample.int(100000, 1)
set.seed(seed)
seed
#84451, 44785

st_intersection(hamilton_net$edges,
             walksheds |> 
  filter(GeoUID == (walksheds_net_attributes |> 
                      filter(motifs >= 378, 
                             transitivity < 0.0207311636201516,
                             Type == "Urban") |> 
                      slice_sample(n=1) |> 
                      pull(GeoUID)))) |>
  ggplot() +
  geom_sf()
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Leaf 3 (suburban):

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc2VlZCA8LSBzYW1wbGUuaW50KDEwMDAwMCwgMSlcbnNldC5zZWVkKHNlZWQpXG5zZWVkXG4jODE0NjEsIDg0NDUxXG5cbnN0X2ludGVyc2VjdGlvbihoYW1pbHRvbl9uZXQkZWRnZXMsXG4gICAgICAgICAgICAgd2Fsa3NoZWRzIHw+IFxuICBmaWx0ZXIoR2VvVUlEID09ICh3YWxrc2hlZHNfbmV0X2F0dHJpYnV0ZXMgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgZmlsdGVyKG1vdGlmcyA8IDEwMjQsIFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICB0cmFuc2l0aXZpdHkgPj0gMC4wMjA3MzExNjM2MjAxNTE2LFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICBUeXBlID09IFwiU3VidXJiYW5cIikgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgc2xpY2Vfc2FtcGxlKG49MSkgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgcHVsbChHZW9VSUQpKSkpIHw+XG4gIGdncGxvdCgpICtcbiAgZ2VvbV9zZigpXG5gYGAifQ== -->

```r
seed <- sample.int(100000, 1)
set.seed(seed)
seed
#81461, 84451

st_intersection(hamilton_net$edges,
             walksheds |> 
  filter(GeoUID == (walksheds_net_attributes |> 
                      filter(motifs < 1024, 
                             transitivity >= 0.0207311636201516,
                             Type == "Suburban") |> 
                      slice_sample(n=1) |> 
                      pull(GeoUID)))) |>
  ggplot() +
  geom_sf()
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Leaf 4 (urban):

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc2VlZCA8LSBzYW1wbGUuaW50KDEwMDAwMCwgMSlcbnNldC5zZWVkKHNlZWQpXG5zZWVkXG4jNTI1MzRcblxuc3RfaW50ZXJzZWN0aW9uKGhhbWlsdG9uX25ldCRlZGdlcyxcbiAgICAgICAgICAgICB3YWxrc2hlZHMgfD4gXG4gIGZpbHRlcihHZW9VSUQgPT0gKHdhbGtzaGVkc19uZXRfYXR0cmlidXRlcyB8PiBcbiAgICAgICAgICAgICAgICAgICAgICBmaWx0ZXIobW90aWZzID49IDEwMjQsIFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICB0cmFuc2l0aXZpdHkgPj0gMC4wMjA3MzExNjM2MjAxNTE2LFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICBUeXBlID09IFwiVXJiYW5cIikgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgc2xpY2Vfc2FtcGxlKG49MSkgfD4gXG4gICAgICAgICAgICAgICAgICAgICAgcHVsbChHZW9VSUQpKSkpIHw+XG4gIGdncGxvdCgpICtcbiAgZ2VvbV9zZigpXG5gYGAifQ== -->

```r
seed <- sample.int(100000, 1)
set.seed(seed)
seed
#52534

st_intersection(hamilton_net$edges,
             walksheds |> 
  filter(GeoUID == (walksheds_net_attributes |> 
                      filter(motifs >= 1024, 
                             transitivity >= 0.0207311636201516,
                             Type == "Urban") |> 
                      slice_sample(n=1) |> 
                      pull(GeoUID)))) |>
  ggplot() +
  geom_sf()
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Add leaf labels to data:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZCA8LSBkIHw+XG4gIG11dGF0ZShsZWFmID0gY2FzZV93aGVuKHRyYW5zaXRpdml0eSA8IDAuMDIwNzMxMTYzNjIwMTUxNiAmIG1vdGlmcyA8IDM3OCB+IFwiTGVhZiAxXCIsXG4gICAgICAgICAgICAgICAgICAgICAgICAgIHRyYW5zaXRpdml0eSA8IDAuMDIwNzMxMTYzNjIwMTUxNiAmIG1vdGlmcyA+PSAzNzggfiBcIkxlYWYgMlwiLFxuICAgICAgICAgICAgICAgICAgICAgICAgICB0cmFuc2l0aXZpdHkgPj0gMC4wMjA3MzExNjM2MjAxNTE2ICYgbW90aWZzIDwgMTAyNCB+IFwiTGVhZiAzXCIsXG4gICAgICAgICAgICAgICAgICAgICAgICAgIHRyYW5zaXRpdml0eSA+PSAwLjAyMDczMTE2MzYyMDE1MTYgJiBtb3RpZnMgPj0gMTAyNCB+IFwiTGVhZiA0XCIpLFxuICAgICAgICAgbGVhZiA9IGZhY3RvcihsZWFmKSlcbmBgYCJ9 -->

```r
d <- d |>
  mutate(leaf = case_when(transitivity < 0.0207311636201516 & motifs < 378 ~ "Leaf 1",
                          transitivity < 0.0207311636201516 & motifs >= 378 ~ "Leaf 2",
                          transitivity >= 0.0207311636201516 & motifs < 1024 ~ "Leaf 3",
                          transitivity >= 0.0207311636201516 & motifs >= 1024 ~ "Leaf 4"),
         leaf = factor(leaf))
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc2tpbShkKVxuYGBgIn0= -->

```r
skim(d)
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZCB8PlxuICBsZWZ0X2pvaW4od2Fsa3NoZWRzIHw+XG4gICAgICAgICAgICAgIHNlbGVjdCgtVHlwZSkgfD5cbiAgICAgICAgICAgICAgbXV0YXRlKGFyZWEgPSBzdF9hcmVhKGdlb21ldHJ5KSxcbiAgICAgICAgICAgICAgICAgICAgIGFyZWEgPSB1bml0czo6c2V0X3VuaXRzKGFyZWEsIGttXjIpIHw+IFxuICAgICAgICAgICAgICAgICAgICAgICBkcm9wX3VuaXRzKCksXG4gICAgICAgICAgICAgICAgICAgICBhcmVhID0gbG9nKGFyZWEpKSB8PlxuICAgICAgICAgICAgICBzdF9kcm9wX2dlb21ldHJ5KCksXG4gICAgICAgICAgICBieSA9IFwiR2VvVUlEXCIpIHw+XG4gIGxtKGFyZWEgfiBsZWFmLCBkYXRhID0gXykgfD5cbiAgc3VtbWFyeSgpXG5gYGAifQ== -->

```r
d |>
  left_join(walksheds |>
              select(-Type) |>
              mutate(area = st_area(geometry),
                     area = units::set_units(area, km^2) |> 
                       drop_units(),
                     area = log(area)) |>
              st_drop_geometry(),
            by = "GeoUID") |>
  lm(area ~ leaf, data = _) |>
  summary()
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


But what about accessibility??

Summary of amenity categories by DA and Type:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuYW1lbml0aWVzX3dhbGtzaGVkc19zdW1tYXJ5IDwtIGFtZW5pdGllc193YWxrc2hlZHMgfD5cbiAgZHJvcF9uYSgpIHw+XG4gIHN0X2Ryb3BfZ2VvbWV0cnkoKSB8PlxuICBncm91cF9ieShHZW9VSUQsXG4gICAgICAgICAgIFR5cGUsXG4gICAgICAgICAgIENhdGVnb3J5KSB8PlxuICBzdW1tYXJpemUobiA9IG4oKSxcbiAgICAgICAgICAgIC5ncm91cHMgPSBcImRyb3BcIikgfD5cbiAgcGl2b3Rfd2lkZXIoaWRfY29scyA9IGMoR2VvVUlELCBUeXBlKSxcbiAgICAgICAgICAgICAgbmFtZXNfZnJvbSA9IENhdGVnb3J5LFxuICAgICAgICAgICAgICB2YWx1ZXNfZnJvbSA9IG4sXG4gICAgICAgICAgICAgIHZhbHVlc19maWxsID0gMClcbmBgYCJ9 -->

```r
amenities_walksheds_summary <- amenities_walksheds |>
  drop_na() |>
  st_drop_geometry() |>
  group_by(GeoUID,
           Type,
           Category) |>
  summarize(n = n(),
            .groups = "drop") |>
  pivot_wider(id_cols = c(GeoUID, Type),
              names_from = Category,
              values_from = n,
              values_fill = 0)
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



To map accessibility by category of amenity we first categorize the number of amenities:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuYW1lbnRpdGllc19kYSA8LSAgdXJiYW5faGFtbWVyX2RhIHw+XG4gIHNlbGVjdChHZW9VSUQsIFR5cGUuMSkgfD5cbiAgcmVuYW1lKFR5cGUgPSBUeXBlLjEpIHw+XG4gIGxlZnRfam9pbihhbWVuaXRpZXNfd2Fsa3NoZWRzX3N1bW1hcnksXG4gICAgICAgICAgICBieSA9IGMoXCJHZW9VSURcIiwgXCJUeXBlXCIpKSB8PlxuICBtdXRhdGUoYWNyb3NzKFN1c3RlbmFuY2U6TGlicmFyeSwgfiByZXBsYWNlX25hKC54LCAwKSkpIHw+XG4gIHBpdm90X2xvbmdlcihjb2xzID0gLWMoR2VvVUlELCBUeXBlLCBnZW9tZXRyeSksXG4gICAgICAgICAgICAgICBuYW1lc190byA9IFwiQW1lbml0eV9DYXRlZ29yeVwiLFxuICAgICAgICAgICAgICAgdmFsdWVzX3RvID0gXCJOdW1iZXJfb2ZfQW1lbml0aWVzXCIpIHw+XG4gICNtdXRhdGUoTnVtYmVyX29mX0FtZW5pdGllcyA9IGlmZWxzZShOdW1iZXJfb2ZfQW1lbml0aWVzID09IDAsIE5BLCBOdW1iZXJfb2ZfQW1lbml0aWVzKSkgfD5cbiAgbXV0YXRlKE51bWJlcl9vZl9BbWVuaXRpZXMgPSBjYXNlX3doZW4oTnVtYmVyX29mX0FtZW5pdGllcyA9PSAwIH4gXCIwXCIsXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIE51bWJlcl9vZl9BbWVuaXRpZXMgPiAwICYgTnVtYmVyX29mX0FtZW5pdGllcyA8PSAzIH4gXCIxLTNcIixcbiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgTnVtYmVyX29mX0FtZW5pdGllcyA+IDMgJiBOdW1iZXJfb2ZfQW1lbml0aWVzIDw9IDYgfiBcIjQtNlwiLFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBOdW1iZXJfb2ZfQW1lbml0aWVzID4gNiAmIE51bWJlcl9vZl9BbWVuaXRpZXMgPD0gOSB+IFwiNy05XCIsXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIE51bWJlcl9vZl9BbWVuaXRpZXMgPiA5IH4gXCIxMCBvciBtb3JlXCIpLFxuICAgICAgICAgTnVtYmVyX29mX0FtZW5pdGllcyA9IGZhY3RvcihOdW1iZXJfb2ZfQW1lbml0aWVzLFxuICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBsZXZlbHMgPSBjKFwiMFwiLCBcIjEtM1wiLCBcIjQtNlwiLCBcIjctOVwiLCBcIjEwIG9yIG1vcmVcIiksXG4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG9yZGVyZWQgPSBUUlVFKSlcbmBgYCJ9 -->

```r
amentities_da <-  urban_hammer_da |>
  select(GeoUID, Type.1) |>
  rename(Type = Type.1) |>
  left_join(amenities_walksheds_summary,
            by = c("GeoUID", "Type")) |>
  mutate(across(Sustenance:Library, ~ replace_na(.x, 0))) |>
  pivot_longer(cols = -c(GeoUID, Type, geometry),
               names_to = "Amenity_Category",
               values_to = "Number_of_Amenities") |>
  #mutate(Number_of_Amenities = ifelse(Number_of_Amenities == 0, NA, Number_of_Amenities)) |>
  mutate(Number_of_Amenities = case_when(Number_of_Amenities == 0 ~ "0",
                                         Number_of_Amenities > 0 & Number_of_Amenities <= 3 ~ "1-3",
                                         Number_of_Amenities > 3 & Number_of_Amenities <= 6 ~ "4-6",
                                         Number_of_Amenities > 6 & Number_of_Amenities <= 9 ~ "7-9",
                                         Number_of_Amenities > 9 ~ "10 or more"),
         Number_of_Amenities = factor(Number_of_Amenities,
                                      levels = c("0", "1-3", "4-6", "7-9", "10 or more"),
                                      ordered = TRUE))
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



Plot:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuYW1lbnRpdGllc19kYSB8PlxuICBnZ3Bsb3QoKSArXG4gIGdlb21fc2YoYWVzKGZpbGwgPSBOdW1iZXJfb2ZfQW1lbml0aWVzKSkgK1xuICBnZW9tX3NmKGRhdGEgPSB1cmJhbl90eXBlcyB8PlxuICAgICAgICAgICAgZmlsdGVyKFR5cGUgIT0gXCJSdXJhbFwiKSxcbiAgICAgICAgICBhZXMoY29sb3IgPSBUeXBlKSxcbiAgICAgICAgICBmaWxsID0gTkEpICtcbiAgI3NjYWxlX2ZpbGxfZmVybWVudGVyKGRpcmVjdGlvbiA9IDEsIHBhbGV0dGUgPSBcIkdyZWVuc1wiKSArXG4gIGZhY2V0X3dyYXAofiBBbWVuaXR5X0NhdGVnb3J5KSArIFxuICB0aGVtZV92b2lkKClcbmBgYCJ9 -->

```r
amentities_da |>
  ggplot() +
  geom_sf(aes(fill = Number_of_Amenities)) +
  geom_sf(data = urban_types |>
            filter(Type != "Rural"),
          aes(color = Type),
          fill = NA) +
  #scale_fill_fermenter(direction = 1, palette = "Greens") +
  facet_wrap(~ Amenity_Category) + 
  theme_void()
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Pivot wider:

<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuanVuayA8LSBhbWVudGl0aWVzX2RhIHw+XG4gIHBpdm90X3dpZGVyKG5hbWVzX2Zyb20gPSBcIkFtZW5pdHlfQ2F0ZWdvcnlcIixcbiAgICAgICAgICAgICAgdmFsdWVzX2Zyb20gPSBcIk51bWJlcl9vZl9BbWVuaXRpZXNcIilcbmBgYCJ9 -->

```r
junk <- amentities_da |>
  pivot_wider(names_from = "Amenity_Category",
              values_from = "Number_of_Amenities")
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->




<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuanVuazIgPC0gZCB8PlxuICBsZWZ0X2pvaW4oanVuayB8PlxuICAgICAgICAgICAgICAjc2VsZWN0KC1UeXBlKSB8PlxuICAgICAgICAgICAgICBzdF9kcm9wX2dlb21ldHJ5KCksXG4gICAgICAgICAgICBieSA9IGMoXCJHZW9VSURcIiwgXCJUeXBlXCIpKSB8PlxuICBtdXRhdGUoYWNyb3NzKFN1c3RlbmFuY2U6TGlicmFyeSwgfmZvcmNhdHM6OmZjdF9kcm9wKC54KSkpXG5gYGAifQ== -->

```r
junk2 <- d |>
  left_join(junk |>
              #select(-Type) |>
              st_drop_geometry(),
            by = c("GeoUID", "Type")) |>
  mutate(across(Sustenance:Library, ~forcats::fct_drop(.x)))
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc3VtbWFyeShqdW5rMiB8PiBzZWxlY3QoU3VzdGVuYW5jZTpMaWJyYXJ5KSlcbmBgYCJ9 -->

```r
summary(junk2 |> select(Sustenance:Library))
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



## Clusters: try self-organizing maps (SOM)


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxud3NuLnNvbSA8LSB0cmFpblNPTSh4LmRhdGEgPSB3YWxrc2hlZHNfbmV0X2F0dHJpYnV0ZXMgfD5cbiAgICAgICAgICAgICAgICAgICAgICAgc2VsZWN0KC1jKEdlb1VJRCwgVHlwZSkpLFxuICAgICAgICAgICAgICAgICAgICAgZGltZW5zaW9uID0gYyg1LDUpLCBcbiAgICAgICAgICAgICAgICAgICAgIHZlcmJvc2UgPSBUUlVFLCBcbiAgICAgICAgICAgICAgICAgICAgIG5iLnNhdmUgPSA1LCBcbiAgICAgICAgICAgICAgICAgICAgIHRvcG8gPSBcImhleGFnb25hbFwiKVxuYGBgIn0= -->

```r
wsn.som <- trainSOM(x.data = walksheds_net_attributes |>
                       select(-c(GeoUID, Type)),
                     dimension = c(5,5), 
                     verbose = TRUE, 
                     nb.save = 5, 
                     topo = "hexagonal")
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxud3NuLnNvbVxuYGBgIn0= -->

```r
wsn.som
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->




<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGxvdCh3c24uc29tLCB3aGF0PVwiZW5lcmd5XCIpXG5gYGAifQ== -->

```r
plot(wsn.som, what="energy")
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->




<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxud3NuLnNvbSRjbHVzdGVyaW5nXG5gYGAifQ== -->

```r
wsn.som$clustering
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxud3NuLnNvbSRjbHVzdGVyaW5nIHw+IHRhYmxlKClcbmBgYCJ9 -->

```r
wsn.som$clustering |> table()
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGxvdCh3c24uc29tLCB3aGF0PVwib2JzXCIsIHR5cGU9XCJoaXRtYXBcIilcbmBgYCJ9 -->

```r
plot(wsn.som, what="obs", type="hitmap")
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc3VtbWFyeSh3c24uc29tKVxuYGBgIn0= -->

```r
summary(wsn.som)
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGFyKG1mcm93ID0gYygyLDIpKVxucGxvdCh3c24uc29tLCB3aGF0ID0gXCJvYnNcIiwgdHlwZSA9IFwiY29sb3JcIiwgdmFyaWFibGUgPSAxKVxucGxvdCh3c24uc29tLCB3aGF0ID0gXCJvYnNcIiwgdHlwZSA9IFwiY29sb3JcIiwgdmFyaWFibGUgPSAyKVxuYGBgIn0= -->

```r
par(mfrow = c(2,2))
plot(wsn.som, what = "obs", type = "color", variable = 1)
plot(wsn.som, what = "obs", type = "color", variable = 2)
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGxvdCh3c24uc29tLCB3aGF0ID0gXCJhZGRcIiwgdHlwZSA9IFwicGllXCIsIHZhcmlhYmxlID0gd2Fsa3NoZWRzX25ldF9hdHRyaWJ1dGVzJFR5cGUpICtcbiAgc2NhbGVfZmlsbF9icmV3ZXIodHlwZSA9IFwicXVhbFwiKSArIFxuICBndWlkZXMoZmlsbCA9IGd1aWRlX2xlZ2VuZCh0aXRsZSA9IFwiVHlwZVwiKSlcbmBgYCJ9 -->

```r
plot(wsn.som, what = "add", type = "pie", variable = walksheds_net_attributes$Type) +
  scale_fill_brewer(type = "qual") + 
  guides(fill = guide_legend(title = "Type"))
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->




<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucXVhbGl0eSh3c24uc29tKVxuYGBgIn0= -->

```r
quality(wsn.som)
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->




<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGxvdChzdXBlckNsYXNzKHdzbi5zb20pKVxuYGBgIn0= -->

```r
plot(superClass(wsn.som))
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->




<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxud3NuLnNvbSA8LSB0cmFpblNPTSh4LmRhdGEgPSB3YWxrc2hlZHNfbmV0X2F0dHJpYnV0ZXMgfD5cbiAgICAgICAgICAgICAgICAgICAgICAgc2VsZWN0KC1jKEdlb1VJRCwgVHlwZSkpLFxuICAgICAgICAgICAgICAgICAgICAgZGltZW5zaW9uID0gYyg0LCA0KSwgIzIsM1xuICAgICAgICAgICAgICAgICAgICAgdmVyYm9zZSA9IFRSVUUsIFxuICAgICAgICAgICAgICAgICAgICAgbmIuc2F2ZSA9IDUsXG4gICAgICAgICAgICAgICAgICAgICB0b3BvID0gXCJzcXVhcmVcIilcbmBgYCJ9 -->

```r
wsn.som <- trainSOM(x.data = walksheds_net_attributes |>
                       select(-c(GeoUID, Type)),
                     dimension = c(4, 4), #2,3
                     verbose = TRUE, 
                     nb.save = 5,
                     topo = "square")
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGxvdCh3c24uc29tLCB3aGF0PVwiZW5lcmd5XCIpXG5gYGAifQ== -->

```r
plot(wsn.som, what="energy")
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucXVhbGl0eSh3c24uc29tKVxuYGBgIn0= -->

```r
quality(wsn.som)
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->









<!-- rnb-text-end -->

