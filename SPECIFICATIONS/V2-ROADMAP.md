# V2 Roadmap: Multi-Platform Ad Change Monitor

## Vision

Extend the Google Ads Change Monitor to support all major advertising platforms, providing unified monitoring and alerting across the entire paid media stack.

## Platform Priority

| Priority | Platform | Rationale |
|----------|----------|-----------|
| 1 | Meta Ads | Highest spend, most complex pixel/CAPI setup |
| 2 | LinkedIn Ads | Growing B2B importance, CAPI integration |
| 3 | TikTok Ads | Emerging platform, pixel stability issues |

## Phase 2.1: Meta Ads Integration

**Key Changes to Detect:**
| Change Type | Severity |
|-------------|----------|
| Pixel removed/disabled | 🔴 Critical |
| CAPI event mapping changed | 🔴 Critical |
| Custom conversion deleted | 🔴 Critical |
| Campaign paused/deleted | 🔴 Critical |
| Budget decreased >50% | 🔴 Critical |
| Budget changes <50% | 🟡 Warning |
| Audience modified | 🟡 Warning |
| Ad creative changes | 🟢 Info |

## Phase 2.2: LinkedIn Ads Integration

**Key Changes to Detect:**
| Change Type | Severity |
|-------------|----------|
| Insight Tag removed | 🔴 Critical |
| Conversion action deleted | 🔴 Critical |
| Campaign paused/deleted | 🔴 Critical |
| Budget decreased >50% | 🔴 Critical |
| Audience targeting changed | 🟡 Warning |
| Bid strategy changed | 🟡 Warning |

## Phase 2.3: TikTok Ads Integration

**Key Changes to Detect:**
| Change Type | Severity |
|-------------|----------|
| Pixel disabled | 🔴 Critical |
| Events API mapping changed | 🔴 Critical |
| Campaign paused/deleted | 🔴 Critical |
| Budget decreased >50% | 🔴 Critical |
| Targeting changes | 🟡 Warning |

## Phase 2.4: Enhanced Notifications

- Email notifications (daily digest)
- Custom webhooks
- Dashboard UI

## Migration Path

1. **No breaking changes** - v1 continues working
2. **Additive configuration** - New platforms opt-in
3. **Unified state** - Migrate from single state to per-platform
4. **Backward compatible** - Old endpoints continue working
