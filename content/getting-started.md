+++
title = 'Getting Started with Crucible'
layout = 'getting-started'
+++

## Make an account

Go to the [Crucible Explorer]({{< link graph_explorer >}}) and sign in to create
an account.

## Install the Python client and CLI

`nano-crucible` provides both the `crucible` command-line tool and the Python
client.

```bash
pip install nano-crucible
```

[PyPI]({{< link pypi >}}) &middot; [GitHub]({{< link github >}}) &middot; [Documentation]({{< link nano_crucible_docs >}})

## Configure the client

This stores your API URL and API key so the CLI and Python client can
authenticate as you.

```bash
crucible config init
```

## Create a project

```bash
crucible project create --project-id <your-project-name>
```

A project name may not contain spaces or punctuation other than hyphens. Run the
command with no flags to be prompted for each field, or supply them up front:

```bash
crucible project create --project-id <your-project-name> -o "LBNL" -e "lead@lbl.gov" \
    --title "Silicon Wafer Study"
```

## Add users to the project

Add everyone you are working with. A user can be named by email, username,
ORCID, or MFID.

```bash
crucible project add-user <your-project-name> --user <user email>
crucible project add-user <your-project-name> --user <username> --role editor
```

Each member should now see the project at the
[Crucible Explorer]({{< link graph_explorer >}}).

## Add an instrument

Register the instrument you plan to upload data from. Running `create` with no
flags prompts for each field.

### Command line

```bash
crucible instrument create

crucible instrument create -n "titanx" --instrument-id titanx --location "B72-201"
```

### Python

`instrument_id`, `instrument_name`, and `location` are required; the rest are
optional.

```python
from crucible import CrucibleClient
from crucible.models import Instrument

client = CrucibleClient()  # reads the config written by `crucible config init`

client.instruments.create(
    Instrument(
        instrument_id="titanx",
        instrument_name="titanx",
        location="B72-201",
        manufacturer="FEI",
        model="Titan 80-300",
        instrument_type="TEM",
    )
)
```

## Create datasets

There are four ways to create a dataset. They all produce the same kind of
record, so use whichever fits your workflow.

{{< options >}}

{{< option title="From the web explorer" open="true" >}}
1. Go to [crucible.lbl.gov/explore]({{< link graph_explorer >}}).
2. Navigate to your project.
3. Choose **New Dataset** in the left-hand side panel.
4. Fill in the information.
5. Click **Create**.
{{< /option >}}

{{< option title="From the command line" >}}
```bash
# Create a dataset record without files
crucible dataset create --project-id <your-project-name> --name "Planned experiment"

# Generic upload (server assigns mfid)
crucible dataset create -i file1.dat file2.csv --project-id <your-project-name>

# Upload with metadata and keywords
crucible dataset create -i data.csv --project-id <your-project-name> \
    --metadata '{"temperature": 300, "pressure": 1.0}' \
    --keywords "experiment,thermal" -m "thermal_analysis"
```
{{< /option >}}

{{< option title="From Python" >}}
```python
from crucible import CrucibleClient
from crucible.models import Dataset

client = CrucibleClient()

dataset = Dataset(
    dataset_name="0105 - 1429 Diffraction 70000 x HAADF 1.44 µm",
    project_id="<your-project-name>",
    instrument_id="titanx",
    measurement="STEM Diffraction EDS",
)

client.datasets.create(
    dataset,
    files=["0105 - 1429 Diffraction 70000 x HAADF 1.44 µm.emd"],
    scientific_metadata={"magnification": 70000, "field_of_view_um": 1.44},
    keywords=["diffraction", "HAADF"],
)
```

`datasets.create()` takes the dataset first, then keyword arguments:

| Argument | Default | Purpose |
| --- | --- | --- |
| `scientific_metadata` | `None` | Dict of metadata attached to the dataset |
| `keywords` | `None` | List of keyword strings |
| `files` | `None` | Local paths to upload, or `AssociatedFile` records for data that lives elsewhere |
| `upload_files` | `True` | Set `False` to catalog local paths in place instead of uploading |
| `ingestor` | `None` | Named ingestor used to parse the files |
| `verbose` | `False` | Print upload progress |
{.table .table-sm}
{{< /option >}}

{{< option title="From a local upload UI" >}}
For instrument computers, the upload UIs give you a drag-and-drop front end.
Instructions for running the app are in the
[repository README]({{< link crucible_upload_uis >}}).

```bash
git clone https://github.com/MolecularFoundryCrucible/crucible-upload-uis.git
```

{{< note title="You will need a service account" >}}
The app runs with a service account API key and permissions. Request one on the
[Discord]({{< link discord >}}) or by emailing [mkwall@lbl.gov](mailto:mkwall@lbl.gov),
then bind it to your instrument:

```bash
crucible instrument bind-sa INSTRUMENT_MFID SERVICE_ACCOUNT_ID
```
{{< /note >}}
{{< /option >}}

{{< /options >}}

## Create samples

There are three ways to create a sample. They all produce the same kind of
record, so use whichever fits your workflow.

{{< options >}}

{{< option title="From the web explorer" open="true" >}}
1. Go to [crucible.lbl.gov/explore]({{< link graph_explorer >}}).
2. Navigate to your project.
3. Choose **New Sample** in the left-hand side panel.
4. Fill in the information.
5. Click **Create**.
{{< /option >}}

{{< option title="From the command line" >}}
```bash
crucible sample create

crucible sample create -n "Silicon Wafer A" --project-id <your-project-name> \
    --description "Test sample" --type substrate
```
{{< /option >}}

{{< option title="From Python" >}}
```python
from crucible.models import Sample

client.samples.create(
    Sample(
        sample_name="Silicon Wafer A",
        project_id="<your-project-name>",
        sample_type="substrate",
        description="Test sample",
    ),
    scientific_metadata={"thickness_um": 525},
)
```
{{< /option >}}

{{< /options >}}

## Create relationships

Records are linked by MFID. Samples and datasets share the same two relationship
types: `is_derived_from` and `is_part_of`.

```python
# sample → sample
client.samples.link(parent_mfid, child_mfid, relationship_type="is_part_of")
client.samples.link(parent_mfid, child_mfid, relationship_type="is_derived_from")

# dataset → dataset
client.datasets.link(parent_mfid, child_mfid, relationship_type="is_part_of")
client.datasets.link(parent_mfid, child_mfid, relationship_type="is_derived_from")

# dataset → sample
client.datasets.link_sample(dataset_mfid, sample_mfid)
```
