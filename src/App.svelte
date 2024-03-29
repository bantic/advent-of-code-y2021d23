<script lang="ts">
  const COSTS = {
    A: 1,
    B: 10,
    C: 100,
    D: 1000,
  };
  let totalCost = 0;
  let selectedIdx = null;
  let plays = [];
  let nodes = makeNodes();

  function selectNode(nodeIdx, record = true) {
    if ((record && canSelectNode(nodeIdx)) || canMoveToNode(nodeIdx)) {
      plays.push(nodeIdx);
    }
    if (canSelectNode(nodeIdx)) {
      selectedIdx = nodeIdx;
    } else if (canMoveToNode(nodeIdx)) {
      handleMove(nodeIdx);
    }
    if (record) {
      console.log("plays", plays);
    }
  }

  function undo() {
    let drops = plays.length % 2 == 0 ? 2 : 1;
    let newPlays = plays.slice(0, -1 * drops);
    totalCost = 0;
    selectedIdx = null;
    nodes = makeNodes();
    for (let nodeIdx of newPlays) {
      selectNode(nodeIdx, false);
    }
    plays = newPlays;
    console.log("plays:", plays);
  }

  function canSelectNode(nodeIdx) {
    return selectedIdx === null && nodes[nodeIdx].occupant !== null;
  }

  function canMoveToNode(nodeIdx) {
    if (selectedIdx === null) {
      return false;
    }
    return canMoveFromNodeToNode(selectedIdx, nodeIdx);
  }

  function handleMove(nodeIdx) {
    let path = canMoveFromNodeToNode(selectedIdx, nodeIdx);
    if (!path) {
      console.log(`handleMove: cannot move to ${nodeIdx}`);
    } else {
      let start = nodes[selectedIdx];
      let end = nodes[nodeIdx];
      end.occupant = start.occupant;
      start.occupant = null;
      selectedIdx = null;
      totalCost += COSTS[end.occupant] * path.length;
      nodes = nodes; // trigger update
    }
    console.log(`moving from ${selectedIdx} -> ${nodeIdx}`);
    selectedIdx = null;
  }

  function pathFrom(aIdx, bIdx, curPath: number[] = []) {
    for (let n of nodes[aIdx].neighbors) {
      if (curPath.includes(n.idx)) {
        continue;
      }
      if (n.occupant !== null) {
        // cannot move into occupied realm
        continue;
      }
      if (n.idx === bIdx) {
        // found
        let copy = curPath.slice();
        copy.push(bIdx);
        return copy;
      }
      let copy = curPath.slice();
      copy.push(aIdx);
      let path = pathFrom(n.idx, bIdx, copy);
      if (path) {
        return path;
      }
    }
    return null;
  }

  function canMoveFromNodeToNode(idxA, idxB) {
    if (idxA === null || idxB === null) {
      throw new Error(
        `invalid params for canMoveFromNodeToNode ${idxA}, ${idxB}`
      );
    }
    let path = pathFrom(idxA, idxB);
    if (!path) {
      return false;
    }
    let occupant = nodes[idxA].occupant;
    if (!nodes[idxB].allowed.includes(occupant)) {
      console.log(
        `cannot move occupant ${occupant} to ${idxB}, it is restricted to ${nodes[idxB].allowed}`
      );
      return false;
    }
    return path;
  }

  function range(n) {
    return Array(n)
      .fill(0)
      .map((_, idx) => idx);
  }

  function makeNodes() {
    let nodes = range(19).map((idx) => {
      return {
        idx,
        neighbors: [],
        allowed: ["A", "B", "C", "D"],
        occupant: null,
      };
    });
    nodes[0].neighbors = [nodes[1]];
    nodes[1].neighbors = [nodes[0], nodes[2]];
    nodes[2].neighbors = [nodes[1], nodes[3], nodes[11]];
    nodes[3].neighbors = [nodes[2], nodes[4]];
    nodes[4].neighbors = [nodes[3], nodes[5], nodes[12]];
    nodes[5].neighbors = [nodes[4], nodes[6]];
    nodes[6].neighbors = [nodes[5], nodes[7], nodes[13]];
    nodes[7].neighbors = [nodes[6], nodes[8]];
    nodes[8].neighbors = [nodes[7], nodes[9], nodes[14]];
    nodes[9].neighbors = [nodes[8], nodes[10]];
    nodes[10].neighbors = [nodes[9]];
    nodes[11].neighbors = [nodes[2], nodes[15]];
    nodes[12].neighbors = [nodes[4], nodes[16]];
    nodes[13].neighbors = [nodes[6], nodes[17]];
    nodes[14].neighbors = [nodes[8], nodes[18]];
    nodes[15].neighbors = [nodes[11]];
    nodes[16].neighbors = [nodes[12]];
    nodes[17].neighbors = [nodes[13]];
    nodes[18].neighbors = [nodes[14]];

    nodes[11].allowed = ["A"];
    nodes[15].allowed = ["A"];
    nodes[12].allowed = ["B"];
    nodes[16].allowed = ["B"];
    nodes[13].allowed = ["C"];
    nodes[17].allowed = ["C"];
    nodes[14].allowed = ["D"];
    nodes[18].allowed = ["D"];

    // cannot stop at top of a hallway
    nodes[2].allowed = [];
    nodes[4].allowed = [];
    nodes[6].allowed = [];
    nodes[8].allowed = [];

    // occupants -- my input
    nodes[11].occupant = "D";
    nodes[15].occupant = "C";
    nodes[12].occupant = "B";
    nodes[16].occupant = "A";
    nodes[13].occupant = "C";
    nodes[17].occupant = "D";
    nodes[14].occupant = "A";
    nodes[18].occupant = "B";

    // occupants -- example
    // nodes[11].occupant = "B";
    // nodes[15].occupant = "A";
    // nodes[12].occupant = "C";
    // nodes[16].occupant = "D";
    // nodes[13].occupant = "B";
    // nodes[17].occupant = "C";
    // nodes[14].occupant = "D";
    // nodes[18].occupant = "A";
    return nodes;
  }

  /**
#############
#...........# -- 0  1   2   3  4   5   6   7  8  9  10
###D#B#C#A### .         11     12      13     14
  #C#A#D#B#             15     16      17     18
  #########

#############
#...........#
###D#B#C#A###
  #C#A#D#B#
  #########
  */
</script>

<main>
  <div id="game">
    <div class="row">
      {#each range(13) as i}
        <div class="wall">#</div>
      {/each}
    </div>
    <div class="row">
      <div class="wall">#</div>
      {#each range(11) as nodeIdx}
        <div
          class="node {selectedIdx === nodeIdx ? 'selected' : ''}"
          on:click={() => selectNode(nodeIdx)}
        >
          {nodes[nodeIdx].occupant || "."}
        </div>
      {/each}
      <div class="wall">#</div>
    </div>
    <div class="row">
      {#each range(3) as i}
        <div class="wall">#</div>
      {/each}
      <div
        class="node {selectedIdx === 11 ? 'selected' : ''}"
        on:click={() => selectNode(11)}
      >
        {nodes[11].occupant || "."}
      </div>
      <div class="wall">#</div>
      <div
        class="node {selectedIdx === 12 ? 'selected' : ''}"
        on:click={() => selectNode(12)}
      >
        {nodes[12].occupant || "."}
      </div>
      <div class="wall">#</div>
      <div
        class="node {selectedIdx === 13 ? 'selected' : ''}"
        on:click={() => selectNode(13)}
      >
        {nodes[13].occupant || "."}
      </div>
      <div class="wall">#</div>
      <div
        class="node {selectedIdx === 14 ? 'selected' : ''}"
        on:click={() => selectNode(14)}
      >
        {nodes[14].occupant || "."}
      </div>
      {#each range(3) as i}
        <div class="wall">#</div>
      {/each}
    </div>
    <div class="row">
      {#each range(2) as i}
        <div class="space">&nbsp;</div>
      {/each}
      <div class="wall">#</div>
      <div class="node" on:click={() => selectNode(15)}>
        {nodes[15].occupant || "."}
      </div>
      <div class="wall">#</div>
      <div class="node" on:click={() => selectNode(16)}>
        {nodes[16].occupant || "."}
      </div>
      <div class="wall">#</div>
      <div class="node" on:click={() => selectNode(17)}>
        {nodes[17].occupant || "."}
      </div>
      <div class="wall">#</div>
      <div class="node" on:click={() => selectNode(18)}>
        {nodes[18].occupant || "."}
      </div>
      <div class="wall">#</div>
      {#each range(2) as i}
        <div class="space">&nbsp;</div>
      {/each}
    </div>
    <div class="row">
      {#each range(2) as i}
        <div class="space">&nbsp;</div>
      {/each}
      {#each range(9) as i}
        <div class="wall">#</div>
      {/each}
      {#each range(2) as i}
        <div class="space">&nbsp;</div>
      {/each}
    </div>
  </div>

  <div>
    <h1>Total Cost: {totalCost}</h1>
  </div>
  <div>
    <button on:click={() => undo()}>Undo</button>
  </div>
</main>

<style>
  main {
    padding: 1em;
    margin: 0 auto;
    font-size: 3em;
  }

  .row {
    display: flex;
    flex-direction: row;
  }

  .node,
  .wall,
  .space {
    width: 2em;
    height: 2em;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #d6d6d6;
    margin: 1px;
  }

  .node {
    cursor: pointer;
  }

  .node.selected {
    background: black;
    color: white;
  }

  .node:not(.selected):hover {
    background: #b1b1b1;
  }
</style>
