---
layout: default
title: R Packages Included in Rocker Version-Stable Docker Images
description: Rocker version-stable Docker images come preloaded with a variety of R packages.  Check out this app to explore which packages are included in the various Rocker images.
categories: app
image: "/assets/img/favicon/favicon-196x196.png"
twitter_share: https://ctt.ac/Db7M0
---

# R Packages Included in Rocker Version-Stable Docker Images

[Rocker](https://www.rocker-project.org) [version-stable Docker images](https://hub.docker.com/r/rocker/r-ver) come preloaded with a variety of R packages.  I wasn't able to find a listing of all the R packages that are installed in the various Rocker images, so I made this little app!  It includes all of the installed R packages in the [r-ver](https://hub.docker.com/r/rocker/r-ver), [rstudio](https://hub.docker.com/r/rocker/rstudio), [tidyverse](https://hub.docker.com/r/rocker/tidyverse), and [verse](https://hub.docker.com/r/rocker/verse) images, versions `3.4.0` to `3.6.1`!  

Try out the search box by typing `tidyverse 3.6.1` to see all of the packages installed in the [tidyverse:3.6.1](https://hub.docker.com/r/rocker/tidyverse) image!  Alternatively, you can filter by image, tag, package, and version using the dropdown boxes at the bottom of the table.

*The table may take a few seconds to load....*
    
## Rocker image R packages

<script src="https://code.jquery.com/jquery-3.4.1.min.js"
    integrity="sha256-CSXorXvZcTkaix6Yvo6HppcZGetbYMGWSFlBw8HfCJo=" crossorigin="anonymous"></script>

<script type="text/javascript" charset="utf8" src="https://cdn.datatables.net/1.10.20/js/jquery.dataTables.js"></script>

<script type="text/javascript">
    $(document).ready(function () {
        $('#rocker-packages').DataTable({
            autoWidth: false,
            columns: [
                { width: "20%" },
                { width: "20%" },
                { width: "40%" },
                { width: "20%" }
            ],
            initComplete: function () {
                this.api().columns().every(function () {
                    var column = this;
                    var select = $('<select><option value=""></option></select>')
                        .appendTo($(column.footer()).empty())
                        .on('change', function () {
                            var val = $.fn.dataTable.util.escapeRegex(
                                $(this).val()
                            );

                            column
                                .search(val ? '^' + val + '$' : '', true, false)
                                .draw();
                        });

                    column.data().unique().sort().each(function (d, j) {
                        select.append('<option value="' + d + '">' + d + '</option>')
                    });
                });
            }
        });
    });</script>

<table id="rocker-packages" class="compact hover stripe order-column">
    <thead>
        <tr>
            <th>Image</th>
            <th>Tag</th>
            <th>Package</th>
            <th>Version</th>
        </tr>
    </thead>

    <tbody>
        {% for package in site.data.rocker_packages %}
        <tr>
            <td>{{ package.Image }}</td>
            <td>{{ package.Tag }}</td>
            <td>{{ package.Package }}</td>
            <td>{{ package.Version }}</td>
        </tr>
        {% endfor %}
    </tbody>

    <tfoot>
        <tr>
            <th>Image</th>
            <th>Tag</th>
            <th>Package</th>
            <th>Version</th>
        </tr>
    </tfoot>

</table>

*To get the installed packages, I pulled a fresh version of each of the images on 2019-10-29.*

<div>
    <hr width="10%">

    {% include call_to_action.html %}

    <a href="/">← Go back</a>
</div>
