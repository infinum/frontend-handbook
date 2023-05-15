Because a Universal application runs the codebase in both the browser and in node, we have to safe-guard access to platform-specific APIs and create abstractions around them.

For example, server-side applications can't reference browser-only global objects such as `window`, `document`, `navigator`, or `location`. Likewise, the part which runs in the browser can not access globals like `process`, or the `fs` API.

Angular provides some injectable abstractions over these objects, such as [Location](https://angular.io/api/common/Location) or [DOCUMENT](https://angular.io/api/common/DOCUMENT); it might substitute adequately for these APIs. If Angular doesn't provide it, it's possible to write new abstractions that delegate to the browser APIs while in the browser and to an alternative implementation while on the server (also known as shimming).

A crucial part in creating these abstractions is the ability to check on which platform the codebase is currently running.

Here is an example of how to provide the `Window` object only on client-side:

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

We can then inject `WINDOW` anywhere where we would like to use it. Note that the reference will be `undefined` on server-side.

```typescript
constructor(@Inject(WINDOW) @Optional() private readonly window?: Window) {}
```

## Platform Service
`PlatformService` is a service used to check which platform the codebase is running in. It is just a simple wrapper around Angular's `PLATFORM_ID` token and `isBrowser` and `isPlatformServer` functions:

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
