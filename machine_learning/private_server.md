# Creating your own datasets on your private server

In addition to querying existing datasets, the QCArchive software can be used to easily generate new ones. In this example, we will create a new dataset of small molecules with up to 3 heavy atoms and compute their DFT and AN1-1x energies. 

For this example, we will use a demonstation "Snowflake" server which runs calculations locally in this Jupyter notebook session. In general, QCArchive can be used with thousands of distributed compute nodes at once. 


```python
import numpy as np
import pandas as pd
import qcportal as ptl

from qcfractal import FractalSnowflakeHandler

server = FractalSnowflakeHandler()
local_client = server.client()
```

Our new dataset will be called "QM3":


```python
qm3 = ptl.collections.Dataset(name="QM3", client=local_client, default_units="hartree")
```

### Adding molecules to a dataset

The following function counts heavy atoms in a molecule:


```python
def count_heavy_atoms(molecule):
    return len(list(filter(lambda a: a != 'H', molecule.symbols)))
```

The `add_entry` function adds a molecule to a dataset. Below, we add all molecules in QM7b with 3 or fewer heavy atoms. The `save` function commits these changes to the server. First, pull QM7b down from the MolSSI server:


```python
client = ptl.FractalClient()
qm7b = client.get_collection("dataset", "QM7b")
qm7b_mols = qm7b.get_molecules()
```


```python
for molecule in qm7b_mols["molecule"]:
    if count_heavy_atoms(molecule) <= 3:
        qm3.add_entry(f"{molecule.name}_{molecule.get_hash()[:2]}", molecule)
qm3.save()
```




    '1'



We can now query the server for the molecules in our dataset:


```python
qm3.get_molecules()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>molecule</th>
    </tr>
    <tr>
      <th>index</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>CH4_b3</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H6_f3</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H3N_a0</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H7N_57</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H4O_07</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H6O_18</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H4O_15</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H6O_69</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H4_96</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H2_66</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C3H6_d1</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C3H8_5c</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C3H6_17</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C3H4_11</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H5N_1a</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
    <tr>
      <th>C2H7N_5e</th>
      <td>Geometry (in Angstrom), charge = 0.0, mult...</td>
    </tr>
  </tbody>
</table>
</div>



And look at one of them:


```python
qm3.get_molecules(subset="C2H6O_18")
```


    _ColormakerRegistry()



    NGLWidget()


### Running calculations

Our QM3 dataset now has all of its molecules, but no properties have been computed for them.


```python
qm3.list_values()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>keywords</th>
      <th>name</th>
    </tr>
    <tr>
      <th>native</th>
      <th>driver</th>
      <th>program</th>
      <th>method</th>
      <th>basis</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



----

<img src="https://raw.githubusercontent.com/psi4/psi4media/master/logos-psi4/psi4square.png" alt="psi4" align="left" style="width: 80px;"/>
<img src="https://raw.githubusercontent.com/aiqm/torchani/master/logo1.png" alt="torchani" align="right" style="width: 120px;"/>

The `compute` function is used to submit calculations for every molecule in a dataset. 
We will compute the ωB97x/6-31g(d) energy for each molecule using the Psi4 program, and the ANI-1x energy using the TorchANI program. (Other supported programs include CFOUR, entos, GAMESS, Q-Chem, Molpro, MOPAC, NWChem, RDKit, TeraChem, and Turbomole.)


```python
qm3.compute(program='psi4', method='wB97x', basis='6-31g(d)')
```




    <ComputeResponse(nsubmitted=16 nexisting=0)>




```python
qm3.compute(program="torchani", method="ANI1x")
```




    <ComputeResponse(nsubmitted=16 nexisting=0)>



The calculations are submitted and run asynchronously.

As before, values are described with `list_values` and queried with `get_values`. Incomplete calculations show up as `NaN`, pandas placeholder for missing data.


```python
qm3.list_values()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>keywords</th>
      <th>name</th>
    </tr>
    <tr>
      <th>native</th>
      <th>driver</th>
      <th>program</th>
      <th>method</th>
      <th>basis</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">True</th>
      <th rowspan="2" valign="top">energy</th>
      <th>psi4</th>
      <th>wb97x</th>
      <th>6-31g(d)</th>
      <td>None</td>
      <td>WB97X/6-31g(d)-Psi4</td>
    </tr>
    <tr>
      <th>torchani</th>
      <th>ani1x</th>
      <th>None</th>
      <td>None</td>
      <td>ANI1X-Torchani</td>
    </tr>
  </tbody>
</table>
</div>




```python
dft_data = qm3.get_values(program='psi4')
dft_data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>WB97X/6-31g(d)-Psi4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>CH4_b3</th>
      <td>-40.4996</td>
    </tr>
    <tr>
      <th>C2H6_f3</th>
      <td>-79.8015</td>
    </tr>
    <tr>
      <th>C2H3N_a0</th>
      <td>-132.714</td>
    </tr>
    <tr>
      <th>C2H7N_57</th>
      <td>-135.122</td>
    </tr>
    <tr>
      <th>C2H4O_07</th>
      <td>-153.746</td>
    </tr>
    <tr>
      <th>C2H6O_18</th>
      <td>-154.99</td>
    </tr>
    <tr>
      <th>C2H4O_15</th>
      <td>-153.786</td>
    </tr>
    <tr>
      <th>C2H6O_69</th>
      <td>-154.979</td>
    </tr>
    <tr>
      <th>C2H4_96</th>
      <td>-78.5573</td>
    </tr>
    <tr>
      <th>C2H2_66</th>
      <td>-77.2971</td>
    </tr>
    <tr>
      <th>C3H6_d1</th>
      <td>-117.863</td>
    </tr>
    <tr>
      <th>C3H8_5c</th>
      <td>-119.106</td>
    </tr>
    <tr>
      <th>C3H6_17</th>
      <td>-117.867</td>
    </tr>
    <tr>
      <th>C3H4_11</th>
      <td>-116.613</td>
    </tr>
    <tr>
      <th>C2H5N_1a</th>
      <td>-133.885</td>
    </tr>
    <tr>
      <th>C2H7N_5e</th>
      <td>-135.13</td>
    </tr>
  </tbody>
</table>
</div>




```python
ml_model_data = qm3.get_values(program='torchani')
ml_model_data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ANI1X-Torchani</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>CH4_b3</th>
      <td>-40.4997</td>
    </tr>
    <tr>
      <th>C2H6_f3</th>
      <td>-79.8012</td>
    </tr>
    <tr>
      <th>C2H3N_a0</th>
      <td>-132.714</td>
    </tr>
    <tr>
      <th>C2H7N_57</th>
      <td>-135.122</td>
    </tr>
    <tr>
      <th>C2H4O_07</th>
      <td>-153.746</td>
    </tr>
    <tr>
      <th>C2H6O_18</th>
      <td>-154.99</td>
    </tr>
    <tr>
      <th>C2H4O_15</th>
      <td>-153.786</td>
    </tr>
    <tr>
      <th>C2H6O_69</th>
      <td>-154.978</td>
    </tr>
    <tr>
      <th>C2H4_96</th>
      <td>-78.5572</td>
    </tr>
    <tr>
      <th>C2H2_66</th>
      <td>-77.2973</td>
    </tr>
    <tr>
      <th>C3H6_d1</th>
      <td>-117.863</td>
    </tr>
    <tr>
      <th>C3H8_5c</th>
      <td>-119.106</td>
    </tr>
    <tr>
      <th>C3H6_17</th>
      <td>-117.867</td>
    </tr>
    <tr>
      <th>C3H4_11</th>
      <td>-116.613</td>
    </tr>
    <tr>
      <th>C2H5N_1a</th>
      <td>-133.885</td>
    </tr>
    <tr>
      <th>C2H7N_5e</th>
      <td>-135.13</td>
    </tr>
  </tbody>
</table>
</div>



We can compare the ANI-1x predictions to the DFT values:


```python
import plotly.express as px

data = pd.merge(dft_data, ml_model_data, left_index=True, right_index=True)

data["Unsigned Difference (Hartree)"] = np.abs(data["WB97X/6-31g(d)-Psi4"] - data["ANI1X-Torchani"])

fig = px.violin(data, y="Unsigned Difference (Hartree)", box=True, 
               title="Difference Distribution between ANI-1x and ωB97x/6-31g(d)")
fig.show()
```


<div>


            <div id="f3b7f349-27cc-4883-9875-cd2243d75bd3" class="plotly-graph-div" style="height:600px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("f3b7f349-27cc-4883-9875-cd2243d75bd3")) {
                    Plotly.newPlot(
                        'f3b7f349-27cc-4883-9875-cd2243d75bd3',
                        [{"alignmentgroup": "True", "box": {"visible": true}, "hoverlabel": {"namelength": 0}, "hovertemplate": "Unsigned Difference (Hartree)=%{y}", "legendgroup": "", "marker": {"color": "#636efa"}, "name": "", "offsetgroup": "", "orientation": "v", "scalegroup": "True", "showlegend": false, "type": "violin", "x0": " ", "xaxis": "x", "y": [0.00010063248149094761, 0.0003092288674935162, 0.0001540255726979467, 0.00012165646592166013, 0.00019515162478001002, 0.0001311025581856029, 0.00019306039604316538, 0.00011784678346771216, 9.726902064244314e-05, 0.00019368579741296799, 0.0005744890448085016, 0.00034002963880652715, 2.8818491770721266e-05, 0.0003052474008597983, 0.0001335049377928499, 0.00013512957684724825], "y0": " ", "yaxis": "y"}],
                        {"height": 600, "legend": {"tracegroupgap": 0}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Difference Distribution between ANI-1x and \u03c9B97x/6-31g(d)"}, "violinmode": "group", "xaxis": {"anchor": "y", "domain": [0.0, 1.0]}, "yaxis": {"anchor": "x", "domain": [0.0, 1.0], "title": {"text": "Unsigned Difference (Hartree)"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('f3b7f349-27cc-4883-9875-cd2243d75bd3');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


## Other calculations you can do

- `Dataset`: Single point, gradients, and freqencies
- `ReactionDataset`: Reactions and interaction energies, with many-body counterpoise corrections
- `OptimizationDataset`: Geometry optimization
- `TorsionDriveDataset`: PES scans over torsion angles, for force field fitting.

Talk to us about adding more!

- (AI)MD trajectories?
- Normal mode sampling?

## The QCArchive stack enables large-scale, multi-resource calculations
![distributed.png](attachment:distributed.png)

## Ongoing data enrichment efforts

With the QCArchive framework, it is very easy to submit new calculations on existing datasets. In the MolSSI database, we are augmenting existing datasets with a common set of calculations at various levels of DFT:

- HF / Def2-TZVP
- LDA / Def2-TZVP
- PBE-D3M(BJ) / Def2-TZVP
- B3LYP-D3M(BJ) / Def2-TZVP
- ωB97x-D3(BJ) / Def2-TZVP

and, where feasible:

- MP2 / cc-pVTZ
- CCSD(T) / cc-pVTZ

Let us know if there are properties/calculations that you would like!

## Extras


```python
from IPython.core.display import HTML

def print_info(dataset):
    print(f"Name: {dataset.data.name}")
    print()
    print(f"Data Points: {dataset.data.metadata['data_points']}")
    print(f"Elements: {dataset.data.metadata['elements']}")
    print(f"Labels: {dataset.data.metadata['labels']}")
    
    display(HTML("<u>Description:</u> " + dataset.data.description))
    
    for cite in dataset.data.metadata["citations"]:
        display(HTML(cite['acs_citation']))
```
