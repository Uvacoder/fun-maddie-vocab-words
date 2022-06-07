<template>
    <div class="min-h-full">
        <div class="bg-indigo-600 pb-32">
            <Disclosure
                as="nav"
                class="bg-indigo-600 border-b border-indigo-300 border-opacity-25 lg:border-none"
                v-slot="{ open }"
            >
                <div class="max-w-7xl mx-auto px-2 sm:px-4 lg:px-8">
                    <div
                        class="relative h-16 flex items-center justify-between lg:border-b lg:border-indigo-400 lg:border-opacity-25"
                    >
                        <div class="px-2 flex items-center lg:px-0">
                            <div class="flex-shrink-0">
                                <LightningBoltIcon
                                    class="h-8 w-8 text-indigo-400"
                                />
                            </div>
                            <div class="hidden lg:block lg:ml-10">
                                <div class="flex space-x-4">
                                    <button
                                        v-for="item in navigation"
                                        :key="item.name"
                                        @click="setActiveItem(item)"
                                        :class="[
                                            item.current
                                                ? 'bg-indigo-700 text-white'
                                                : 'text-white hover:bg-indigo-500 hover:bg-opacity-75',
                                            'rounded-md py-2 px-3 text-sm font-medium',
                                        ]"
                                        :aria-current="
                                            item.current ? 'page' : undefined
                                        "
                                    >
                                        {{ item.name }}
                                    </button>
                                </div>
                            </div>
                        </div>
                        <div class="flex lg:hidden">
                            <!-- Mobile menu button -->
                            <DisclosureButton
                                class="bg-indigo-600 p-2 rounded-md inline-flex items-center justify-center text-indigo-200 hover:text-white hover:bg-indigo-500 hover:bg-opacity-75 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-indigo-600 focus:ring-white"
                            >
                                <span class="sr-only">Open main menu</span>
                                <MenuIcon
                                    v-if="!open"
                                    class="block h-6 w-6"
                                    aria-hidden="true"
                                />
                                <XIcon
                                    v-else
                                    class="block h-6 w-6"
                                    aria-hidden="true"
                                />
                            </DisclosureButton>
                        </div>
                    </div>
                </div>

                <DisclosurePanel class="lg:hidden">
                    <div class="px-2 pt-2 pb-3 space-y-1">
                        <DisclosureButton
                            v-for="item in navigation"
                            :key="item.name"
                            @click="setActiveItem(item)"
                            :class="[
                                item.current
                                    ? 'bg-indigo-700 text-white'
                                    : 'text-white hover:bg-indigo-500 hover:bg-opacity-75',
                                'block rounded-md py-2 px-3 text-base font-medium',
                            ]"
                            :aria-current="item.current ? 'page' : undefined"
                        >
                            {{ item.name }}
                        </DisclosureButton>
                    </div>
                </DisclosurePanel>
            </Disclosure>
            <header class="py-10">
                <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                    <h1 class="text-3xl font-bold text-white">
                        {{ activeItem.name }}
                    </h1>
                </div>
            </header>
        </div>

        <main class="-mt-32">
            <div class="max-w-7xl mx-auto pb-12 px-4 sm:px-6 lg:px-8">
                <div class="bg-white rounded-lg shadow px-5 py-6 sm:px-6">
                    <div class="h-96 rounded-lg flex flex-col">
                        <div
                            class="flex justify-end"
                            v-if="gameStatus === 'playing'"
                        >
                            <button
                                class="inline-flex items-center px-2.5 py-1.5 border border-gray-300 shadow-sm text-xs font-medium rounded text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
                                @click="resetGame"
                            >
                                Reset
                            </button>
                        </div>
                        <div class="flex-1 flex items-center justify-center">
                            <button
                                v-if="gameStatus === 'not-started'"
                                class="inline-flex items-center px-8 py-4 border border-transparent text-xl font-medium rounded-lg shadow-sm text-white bg-emerald-600 hover:bg-emerald-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-emerald-500"
                                @click="startGame"
                            >
                                Play!
                            </button>
                            <div
                                v-else-if="gameStatus === 'done'"
                                class="text-center"
                            >
                                <div
                                    class="mb-8 text-6xl text-emerald-700 font-black"
                                >
                                    Done!
                                </div>
                                <button
                                    class="inline-flex items-center px-4 py-2 border border-gray-300 shadow-sm text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
                                    @click="resetGame"
                                >
                                    Play again
                                </button>
                            </div>
                            <button
                                v-else
                                class="font-bold text-6xl h-full w-full inline-flex items-center justify-center"
                                @click="nextWord"
                            >
                                {{ currentWord }}
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </main>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { Disclosure, DisclosureButton, DisclosurePanel } from '@headlessui/vue'
import { MenuIcon, XIcon } from '@heroicons/vue/outline'
import { LightningBoltIcon } from '@heroicons/vue/solid'
import words from './constants/snap-words'

const navigation = ref([
    { name: 'All', current: true },
    { name: 'Kindergarten', current: false },
    { name: 'First', current: false },
])

// active nav item
const activeItem = ref(navigation.value[0])

onMounted(() => {
    setActiveItem(navigation.value[0])
})
// game control setup
// we'll have a status (not-started, playing, done)
// we'll keep track of words we've seen in this current game
// any time we change the word list, we'll reset the seenWords
const gameStatus = ref('not-started')
let seenWords = new Set()
const currentWord = ref('')
const activeWordList = ref([])

function setActiveItem(item) {
    activeItem.value = item
    setActiveNav(item)
    setSnapWords(item)
}

function setActiveNav(item) {
    for (let n of navigation.value) {
        if (n.name === item.name) {
            n.current = true
        } else {
            n.current = false
        }
    }
}

function setSnapWords() {
    // load snap words for current item
    // reset game play status
    gameStatus.value = 'not-started'
    currentWord.value = ''
    seenWords = new Set()
    let key = activeItem.value.name.toLowerCase()
    if (key === 'all') {
        activeWordList.value = Array.from(
            new Set([...words.kindergarten, ...words.first])
        )
    } else {
        activeWordList.value = [...words[key]]
    }
}

function startGame() {
    console.log('start game')
    currentWord.value = getNextWord()
    gameStatus.value = 'playing'
}

function resetGame() {
    // this will essentially reset the game with the same word list.
    setActiveItem(activeItem.value)
}

function nextWord() {
    let next = getNextWord()
    if (next) {
        currentWord.value = next
        seenWords.add(next)
    } else {
        // if we don't get a next word, the game is done.
        gameStatus.value = 'done'
    }
}

function getNextWord() {
    let availableWords = activeWordList.value.filter((w) => !seenWords.has(w))
    if (!availableWords.length) {
        return null
    }
    return availableWords[Math.floor(Math.random() * availableWords.length)]
}
</script>
