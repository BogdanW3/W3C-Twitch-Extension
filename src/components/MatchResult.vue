<template>
  <div v-if="hero && opponent" class="match-result">
    <span v-if="withOpponentName" style="color: var(--color-yellow)">
      {{ playerAkas[opponent.battleTag] || opponent.name }}
      <span v-if="playerAkas[opponent.battleTag]">as {{ opponent.name }}</span>
    </span>
    on {{ getMapName(match.map) }} in
    {{ formatMatchDuration(match.durationInSeconds) }}
    &rarr;
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { Match, PlayerInTeam } from "@/typings";
import { mapNames } from "@/constants/constants";
import intervalToDuration from "date-fns/intervalToDuration";
import usePlayerAka from "@/composables/usePlayerAka";

function formatMatchDuration(interval: number): string {
  const duration = intervalToDuration({ start: 0, end: interval * 1000 });
  const durations = [duration.minutes, duration.seconds];

  if (duration.hours && duration.hours > 0) durations.unshift(duration.hours);

  return durations.map(duration => String(duration).padStart(2, "0")).join(":");
}

function getMapName(mapName: string) {
  const translatedName = mapNames[mapName as keyof typeof mapNames];
  if (translatedName) return translatedName;
  return mapName;
}

export default defineComponent({
  name: "MatchResult",
  props: {
    battleTag: {
      type: String,
      required: true
    },
    match: {
      type: Object as () => Match,
      required: true
    },
    withOpponentName: {
      type: Boolean,
      default: true
    }
  },
  async setup(props) {
    const nullPlayer: PlayerInTeam = {
      oldMmr: 0,
      currentMmr: 0,
      battleTag: "",
      name: "",
      mmrGain: 0,
      race: 0,
      won: false
    };
    let hero: PlayerInTeam = nullPlayer;
    let opponent: PlayerInTeam = nullPlayer;
    const { fetchPlayerAka, playerAkas } = usePlayerAka();
    const fetchAkaPromises = [];

    for (const team of props.match.teams) {
      for (const player of team.players) {
        if (player.battleTag === props.battleTag) {
          hero = player;
        } else {
          opponent = player;
        }

        fetchAkaPromises.push(fetchPlayerAka(player.battleTag));
      }
    }

    await Promise.all(fetchAkaPromises);

    return {
      hero,
      opponent,
      getMapName,
      formatMatchDuration,
      playerAkas
    };
  }
});
</script>

<style lang="scss" scoped>
.match-result {
  cursor: pointer;
}
</style>
