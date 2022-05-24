Because a Universal application doesn't execute in the browser, some browser APIs and capabilities might be missing on the server.

For example, server-side applications can't reference browser-only global objects such as `window`, `document`, `navigator`, or `location`.

Angular provides some injectable abstractions over these objects, such as [Location](https://angular.io/api/common/Location) or [DOCUMENT](https://angular.io/api/common/DOCUMENT); it might substitute adequately for these APIs. If Angular doesn't provide it, it's possible to write new abstractions that delegate to the browser APIs while in the browser and to an alternative implementation while on the server (also known as shimming).

To fix this we can explicitly check for the platform and provide correct value.

In the `app.module.ts` we can check for platform when providing injection token:
```typescript
{
    provide: WINDOW,
        useFactory: (platform: PlatformService) => {
        if (!platform.isBrowser) {
            return undefined;
        }

        return window;
    },
        deps: [PlatformService]
}
```

And then when injecting:

```typescript
constructor(@Inject(WINDOW) @Optional() private readonly window?: Window) {}
```

## Platform Service
This is service used to check the type  of the platform.

```typescript
@Injectable({
	providedIn: 'root',
})
export class PlatformService {
	constructor(@Inject(PLATFORM_ID) private readonly platformId: Record<string, any>) {}

	public get isBrowser(): boolean {
		return isPlatformBrowser(this.platformId);
	}

	public get isServer(): boolean {
		return isPlatformServer(this.platformId);
	}
}
```
