<script>
  import { onMount } from "svelte";
  import { authenticated } from "../stores/auth";
  import { goto } from "@sapper/app.mjs";
  import {
    Button,
    Modal,
    ModalBody,
    ModalFooter,
    ModalHeader,
  } from "sveltestrap";
  let open = false;
  const toggle = () => (open = !open);
  let token = "";
  let dataList = [];
  onMount(async () => {
    try {
      authenticated.subscribe(async (value) => {
        if (value) {
          await fetch("http://localhost:4000/api/inventory/user", {
            method: "GET",
            headers: {
              Accept: "application/json",
              "Content-Type": "application/x-www-form-urlencoded",
              Authorization: `${value}`,
            },
          })
            .then((response) => response.json())
            .then(async (responseJson) => {
              if (responseJson.metadata.http_code == "200") {
                dataList = responseJson.data;
                token = value;
              }
            })
            .catch((error) => {
              console.error(error);
              token = "";
            });
        } else {
          window.location.href = "/login";
        }
      });
    } catch (e) {
      authenticated.set("");
      token = "";
      window.location.href = "/login";
    }
  });

  const editData = (id) => {
    goto(`/barang/${id}`);
  };

  const deleteData = async (id) => {
    authenticated.subscribe(async (value) => {
      if (value) {
        await fetch(
          "http://localhost:4000/api/inventory/user/" + id,
          {
            method: "DELETE",
            headers: {
              Accept: "application/json",
              "Content-Type": "application/x-www-form-urlencoded",
              Authorization: `${value}`,
            },
          }
        )
          .then((response) => response.json())
          .then(async (responseJson) => {
            if (responseJson.metadata.http_code == "200") {
              alert("Data berhasil dihapus");
              toggle();
              goto("/barang");
            }
          })
          .catch((error) => {
            alert("Data gagal dihapus");
            toggle();
          });
      }
    });
  };
</script>

{#if token != ""}
  <div class="container-fluid">
    <div class="row">
      <div class="col">
        <nav aria-label="breadcrumb">
          <ol class="breadcrumb">
            <li class="breadcrumb-item"><a href="/">Home</a></li>
            <li class="breadcrumb-item">
              <a href="/Subbarang">user</a>
            </li>
          </ol>
        </nav>
      </div>
    </div>

    <table class="table table-striped">
      <thead>
        <tr>
          <th scope="col">Nama</th>
          <th scope="col">Email</th>
          <th scope="col">Action</th>
        </tr>
      </thead>
      <tbody>
        {#each dataList as data}
          <tr>
            <td>{data.name}</td>
            <td>{data.email != undefined ? data.email : "-"}</td>
            <td><Button color="danger" on:click={toggle}>Hapus</Button></td>
            <td><Button color="primary" on:click={editData(data.id)}>Edit</Button></td>
            <td><a href="/inventory_input">
              <button type="button" class="btn btn-primary">Tambah</button>
            </a></td>
          </tr>
          <Modal isOpen={open} {toggle}>
            <ModalHeader {toggle}>Hapus Data</ModalHeader>
            <ModalBody>Apakah anda yakin akan menghapus data?</ModalBody>
            <ModalFooter>
              <Button color="danger" on:click={deleteData(data.id)}
                >Hapus</Button
              >
              <Button color="secondary" on:click={toggle}>Cancel</Button>
            </ModalFooter>
          </Modal>
        {/each}
      </tbody>
    </table>
  </div>
{/if}
