# Quest: Scaffold Emoji Token Market — `craft.awizard.dev`

> **Assigned to:** Token wizard (this chat)  
> **Perps wizard is:** building trading interface in `projects/chia-perps/`  
> **Zero overlap:** All work is in new `projects/chia-craft/` — completely different project and domain

---

## Objective

Create the emoji token market system on `craft.awizard.dev` — a fun token creation interface that lets users mint the 6 core emoji CATs and launch them on the CFMM. This demonstrates the creative/social side of the aWizard DeFi ecosystem.

**Time estimate:** 40 minutes — perfect parallel quest!

---

## Context

From `TODO_DEFI.md` Phase 5, the emoji market is a key piece of the DeFi ecosystem:

**6 Core Emoji Tokens:**
- ❤️ **LOVE** — Love / social capital  
- 🌱 **SPROUT** — Growth / ecological
- 🔮 **CASTER** — Magic / creative power
- ✨ **SPELL** — Utility / action fuel
- ⚡ **POWER** — Energy / compute  
- 💎 **HODL** — Store of value / conviction

This creates a **token creation interface** that complements the existing AMM (forge.awizard.dev) and treasure chest (storefront) infrastructure.

---

## Steps

### Step 1 — Project Scaffold

Create new `projects/chia-craft/` from the proven template:

```bash
# Copy from chia-cfmm template
robocopy projects/chia-cfmm projects/chia-craft /E /XD node_modules dist .git /XF "*.lock" /NP /NFL /NDL

# Update package.json
# Set name: "chia-craft"
# Set port to 5176 in dev script
```

### Step 2 — Core Token Definitions  

Create `src/lib/emojiTokens.ts`:

```typescript
export interface EmojiToken {
  id: string;
  emoji: string;
  name: string;
  symbol: string;
  description: string;
  category: 'social' | 'growth' | 'magic' | 'utility' | 'energy' | 'store';
  color: string; // Hex color for UI theming
  maxSupply?: string; // Optional supply cap
  metadata: {
    backstory: string;
    useCase: string;
    emotion: string;
  };
}

export const CORE_EMOJI_TOKENS: EmojiToken[] = [
  {
    id: 'love',
    emoji: '❤️',
    name: 'Love Token',
    symbol: 'LOVE',
    description: 'Social capital representing love and community bonds',
    category: 'social',
    color: '#FF1744',
    metadata: {
      backstory: 'Born from the first hug between wizards',
      useCase: 'Rewards for community building and positive interactions',
      emotion: 'Love, connection, warmth'
    }
  },
  // ... other 5 tokens
];
```

### Step 3 — Token Creation UI

Create `src/components/TokenCreator.tsx`:

```tsx
interface TokenCreatorProps {
  selectedToken?: EmojiToken;
  onMint: (token: EmojiToken, amount: string) => void;
}

export function TokenCreator({ selectedToken, onMint }: TokenCreatorProps) {
  // Token selection grid (6 emoji buttons)
  // Mint amount input with XCH cost calculation  
  // Animated mint button with spell-casting theme
  // Preview of token metadata (backstory, use case)
  
  return (
    <div className="glow-card p-6">
      <h2 className="glow-text text-2xl mb-4">🧙‍♂️ Token Conjuring Chamber</h2>
      {/* Emoji token grid */}
      {/* Amount input */} 
      {/* Mint spell button */}
    </div>
  );
}
```

### Step 4 — Emoji Token Grid

Create `src/components/EmojiTokenGrid.tsx`:

```tsx
export function EmojiTokenGrid({ tokens, selectedToken, onSelect }) {
  return (
    <div className="grid grid-cols-3 gap-4">
      {tokens.map(token => (
        <button
          key={token.id}
          className={`glow-card p-4 text-center transition-all hover:scale-105 ${
            selectedToken?.id === token.id ? 'ring-2 ring-accent' : ''
          }`}
          onClick={() => onSelect(token)}
          style={{ borderColor: token.color }}
        >
          <div className="text-4xl mb-2">{token.emoji}</div>
          <div className="font-semibold">{token.symbol}</div>
          <div className="text-sm text-muted">{token.category}</div>
        </button>
      ))}
    </div>
  );
}
```

### Step 5 — Token Metadata Display

Create `src/components/TokenMetadata.tsx`:

```tsx
export function TokenMetadata({ token }: { token: EmojiToken }) {
  return (
    <div className="glow-card p-6">
      <div className="flex items-center gap-4 mb-4">
        <div className="text-6xl">{token.emoji}</div>
        <div>
          <h3 className="glow-text text-2xl font-bold">{token.name}</h3>
          <p className="text-accent font-mono">${token.symbol}</p>
        </div>
      </div>
      
      <div className="space-y-4">
        <div>
          <h4 className="font-semibold text-primary mb-2">Backstory</h4>
          <p className="text-secondary">{token.metadata.backstory}</p>
        </div>
        
        <div>
          <h4 className="font-semibold text-primary mb-2">Use Case</h4>
          <p className="text-secondary">{token.metadata.useCase}</p>
        </div>
        
        <div>
          <h4 className="font-semibold text-primary mb-2">Emotion</h4>
          <p className="text-warning">{token.metadata.emotion}</p>
        </div>
      </div>
    </div>
  );
}
```

### Step 6 — Main App Integration

Update `src/App.tsx` to showcase the emoji token system:

```tsx
function App() {
  const [selectedToken, setSelectedToken] = useState<EmojiToken | null>(null);
  const [mintAmount, setMintAmount] = useState('');
  
  return (
    <div className="min-h-screen bg-deep p-6">
      <div className="max-w-6xl mx-auto">
        <header className="text-center mb-8">
          <h1 className="glow-text text-4xl font-bold mb-2">
            🎭 craft.awizard.dev
          </h1>
          <p className="text-secondary">
            Conjure emoji tokens and launch them into the DeFi ecosystem
          </p>
        </header>
        
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
          <div>
            <EmojiTokenGrid 
              tokens={CORE_EMOJI_TOKENS}
              selectedToken={selectedToken}
              onSelect={setSelectedToken}
            />
            
            {selectedToken && (
              <TokenCreator 
                selectedToken={selectedToken}
                onMint={handleMint}
              />
            )}
          </div>
          
          <div>
            {selectedToken && <TokenMetadata token={selectedToken} />}
          </div>
        </div>
      </div>
    </div>
  );
}
```

### Step 7 — Environment Configuration

Update `.env.example`:
```bash
# Wallet Connect v2
VITE_WC_PROJECT_ID=your_walletconnect_project_id_here

# Chia Network 
VITE_CHIA_NETWORK=testnet11

# Development URLs
VITE_CRAFT_API_URL=http://localhost:3003
VITE_FORGE_URL=http://localhost:5173
```

---

## Success Criteria

- [ ] `projects/chia-craft/` scaffolded from chia-cfmm template
- [ ] Core 6 emoji tokens defined with full metadata
- [ ] `EmojiTokenGrid.tsx` component with interactive selection
- [ ] `TokenCreator.tsx` component with mint interface
- [ ] `TokenMetadata.tsx` component showing token details  
- [ ] App integrated with Nightspire theme consistency
- [ ] TypeScript build passes with no errors
- [ ] Dev server runs on `localhost:5176`

**Completion unlocks:** CAT launcher contract integration, CFMM pool creation, metadata standards ⚡